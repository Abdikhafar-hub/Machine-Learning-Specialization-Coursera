zzzzzbbvbvbvmmmnmm mknknjjhjjkjjnjhmmmmmmmn n nkmnmkommmmmmqqqqqbbnnnmmmmsssssssssmmmsssssssnssddddddsssssssssaaaaadddddddmssmmmmmmmmmmmmmmmmmmhhhhhhhhaaaaaaaaaasssssssmmmmmmmmn fnnnmdddddmccvcbvghuyk<<<<<<<zz
zzaaaaaaaaa<<<<<<<<<<<<<<<<<<<<<<Create the profiles sssn m
CREATE TABLE profiles ( 
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  name TEXT NOT NULL, m
  email TEXT NOT NULL,
  avatar_url TEXT,
  company TEXT,
  job_title TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,n
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMPn 
);

-- Enable Row Level Security
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- Create policy to allow users to view and modify only their own profile
CREATE POLICY "Users can view their own profile" 
  ON profiles 
  FOR SELECT 
  USING (auth.uid() = id);

CREATE POLICY "Users can update their own profile" 
  ON profilesg
  FOR UPDATE 
  USING (auth.uid() = id);

CREATE POLICY "Users can insert their own profile" 
  ON profiles 
  FOR INSERT 
  WITH CHECK (auth.uid() = id);

-- Create trigger to update the updated_at column
CREATE OR REPLACE FUNCTION update_modified_column() 
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = CURRENT_TIMESTAMP;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_profiles_updated_at
  BEFORE UPDATE ON profiles
  FOR EACH ROW
  EXECUTE PROCEDURE update_modified_column();

-- 2. Create the contacts table
CREATE TABLE contacts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  email TEXT NOT NULL,
  phone TEXT,
  company TEXT,
  website TEXT,
  linkedinUrl TEXT,
  notes TEXT,
  synergy TEXT,
  last_contact TIMESTAMP WITH TIME ZONE,
  follow_up_date TIMESTAMP WITH TIME ZONE,
  email_status TEXT CHECK (email_status IN ('draft', 'scheduled', 'sent', 'failed')),
  email_content TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Enable Row Level Security
ALTER TABLE contacts ENABLE ROW LEVEL SECURITY;

-- Create policies for contacts
CREATE POLICY "Users can view their own contacts" 
  ON contacts 
  FOR SELECT 
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own contacts" 
  ON contacts 
  FOR INSERT 
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own contacts" 
  ON contacts 
  FOR UPDATE 
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete their own contacts" 
  ON contacts 
  FOR DELETE 
  USING (auth.uid() = user_id);

-- Create trigger for contacts updated_at
CREATE TRIGGER update_contacts_updated_at
  BEFORE UPDATE ON contacts
  FOR EACH ROW
  EXECUTE PROCEDURE update_modified_column();

-- 3. Create the follow_ups table
CREATE TABLE follow_ups (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contact_id UUID NOT NULL REFERENCES contacts(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  scheduled_date TIMESTAMP WITH TIME ZONE NOT NULL,
  content TEXT NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('draft', 'scheduled', 'sent', 'failed')),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Enable Row Level Security
ALTER TABLE follow_ups ENABLE ROW LEVEL SECURITY;

-- Create policies for follow_ups
CREATE POLICY "Users can view their own follow_ups" 
  ON follow_ups 
  FOR SELECT 
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own follow_ups" 
  ON follow_ups 
  FOR INSERT 
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own follow_ups" 
  ON follow_ups 
  FOR UPDATE 
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete their own follow_ups" 
  ON follow_ups 
  FOR DELETE 
  USING (auth.uid() = user_id);

-- Create trigger for follow_ups updated_at
CREATE TRIGGER update_follow_ups_updated_at
  BEFORE UPDATE ON follow_ups
  FOR EACH ROW
  EXECUTE PROCEDURE update_modified_column();

-- 4. Create the email_templates table
CREATE TABLE email_templates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  content TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Enable Row Level Security
ALTER TABLE email_templates ENABLE ROW LEVEL SECURITY;

-- Create policies for email_templates
CREATE POLICY "Users can view their own email_templates" 
  ON email_templates 
  FOR SELECT 
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own email_templates" 
  ON email_templates 
  FOR INSERT 
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own email_templates" 
  ON email_templates 
  FOR UPDATE 
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete their own email_templates" 
  ON email_templates 
  FOR DELETE 
  USING (auth.uid() = user_id);

-- Create trigger for email_templates updated_at
CREATE TRIGGER update_email_templates_updated_at
  BEFORE UPDATE ON email_templates
  FOR EACH ROW
  EXECUTE PROCEDURE update_modified_column();

-- 5. Create useful views

-- View to show upcoming follow-ups
CREATE VIEW upcoming_follow_ups AS
SELECT fu.*, c.name as contact_name, c.email as contact_email
FROM follow_ups fu
JOIN contacts c ON fu.contact_id = c.id
WHERE fu.status = 'scheduled' AND fu.scheduled_date > CURRENT_TIMESTAMP
ORDER BY fu.scheduled_date ASC;

-- View to show contacts without recent follow-ups
CREATE VIEW contacts_needing_followup AS
SELECT c.*
FROM contacts c
LEFT JOIN (
  SELECT contact_id, MAX(scheduled_date) as last_followup_date
  FROM follow_ups
  WHERE status IN ('sent', 'scheduled')
  GROUP BY contact_id
) fu ON c.id = fu.contact_id
WHERE fu.contact_id IS NULL OR (CURRENT_DATE - fu.last_followup_date::date) > 30
ORDER BY COALESCE(fu.last_followup_date, c.created_at) ASC;

-- 6. Create indexes for better performance
CREATE INDEX contacts_user_id_idx ON contacts(user_id);
CREATE INDEX follow_ups_user_id_idx ON follow_ups(user_id);
CREATE INDEX follow_ups_contact_id_idx ON follow_ups(contact_id);
CREATE INDEX follow_ups_scheduled_date_idx ON follow_ups(scheduled_date);
CREATE INDEX email_templates_user_id_idx ON email_templates(user_id);








![](/resources/title-head.png)

Contains Solutions and Notes for the [Machine Learning Specialization](https://www.coursera.org/specializations/machine-learning-introduction/?utm_medium=coursera&utm_source=home-page&utm_campaign=mlslaunch2022IN) by Andrew NG on Coursera 

**Note : If you would like to have a deeper understanding of the concepts by understanding all the math required, have a look at [Mathematics for Machine Learning and Data Science](https://github.com/greyhatguy007/Mathematics-for-Machine-Learning-and-Data-Science-Specialization-Coursera)**

<hr/>

## Course 1 : [Supervised Machine Learning: Regression and Classification ](https://www.coursera.org/learn/machine-learning?specialization=machine-learning-introduction)

- [Week 1](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week1)

    - [Practice quiz: Regression](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week1/Practice%20quiz%20-%20Regression)
    - [Practice quiz: Supervised vs unsupervised learning](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week1/Practice%20quiz%20-%20Supervised%20vs%20unsupervised%20learning)
    - [Practice quiz: Train the model with gradient descent](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week1/Practice%20quiz%20-%20Train%20the%20model%20with%20gradient%20descent)
  - [Optional Labs](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week1/Optional%20Labs)
    - [Model Representation](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week1/Optional%20Labs/C1_W1_Lab03_Model_Representation_Soln.ipynb)
    - [Cost Function](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week1/Optional%20Labs/C1_W1_Lab04_Cost_function_Soln.ipynb)
    - [Gradient Descent](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week1/Optional%20Labs/C1_W1_Lab05_Gradient_Descent_Soln.ipynb)

<br/>

- [Week 2](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week2) 

    - [Practice quiz: Gradient descent in practice](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week2/Practice%20quiz%20-%20Gradient%20descent%20in%20practice)
    - [Practice quiz: Multiple linear regression](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week2/Practice%20quiz%20-%20Multiple%20linear%20regression)
    - [Optional Labs](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week2/Optional%20Labs)
      - [Numpy Vectorization](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week2/Optional%20Labs/C1_W2_Lab01_Python_Numpy_Vectorization_Soln.ipynb)
      - [Multi Variate Regression](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week2/Optional%20Labs/C1_W2_Lab02_Multiple_Variable_Soln.ipynb)
      - [Feature Scaling](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week2/Optional%20Labs/C1_W2_Lab03_Feature_Scaling_and_Learning_Rate_Soln.ipynb)
      - [Feature Engineering](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week2/Optional%20Labs/C1_W2_Lab04_FeatEng_PolyReg_Soln.ipynb)
      - [Sklearn Gradient Descent](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week2/Optional%20Labs/C1_W2_Lab05_Sklearn_GD_Soln.ipynb)
      - [Sklearn Normal Method](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week2/Optional%20Labs/C1_W2_Lab05_Sklearn_GD_Soln.ipynb)
    - [Programming Assignment](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week2/C1W2A1)
      - [Linear Regression](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week2/C1W2A1/C1_W2_Linear_Regression.ipynb)

<br/>

- [Week 3](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3)

    - [Practice quiz: Cost function for logistic regression](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3/Practice%20quiz%20-%20Cost%20function%20for%20logistic%20regression)
    - [Practice quiz: Gradient descent for logistic regression](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3/Practice%20quiz%20-%20Gradient%20descent%20for%20logistic%20regression)
    - [Optional Labs](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3/Optional%20Labs)
        - [Classification](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3/Optional%20Labs/C1_W3_Lab01_Classification_Soln.ipynb)
        - [Sigmoid Function](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3/Optional%20Labs/C1_W3_Lab02_Sigmoid_function_Soln.ipynb)
        - [Decision Boundary](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3/Optional%20Labs/C1_W3_Lab03_Decision_Boundary_Soln.ipynb)
        - [Logistic Loss](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3/Optional%20Labs/C1_W3_Lab04_LogisticLoss_Soln.ipynb)
        - [Cost Function](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3/Optional%20Labs/C1_W3_Lab05_Cost_Function_Soln.ipynb)
        - [Gradient Descent](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3/Optional%20Labs/C1_W3_Lab06_Gradient_Descent_Soln.ipynb)
        - [Scikit Learn - Logistic Regression](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3/Optional%20Labs/C1_W3_Lab07_Scikit_Learn_Soln.ipynb)
        - [Overfitting](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3/Optional%20Labs/C1_W3_Lab08_Overfitting_Soln.ipynb)
        - [Regularization](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3/Optional%20Labs/C1_W3_Lab09_Regularization_Soln.ipynb)
    - [Programming Assignment](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3/C1W3A1)
      - [Logistic Regression](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C1%20-%20Supervised%20Machine%20Learning%20-%20Regression%20and%20Classification/week3/C1W3A1/C1_W3_Logistic_Regression.ipynb)

#### [Certificate Of Completion](https://coursera.org/share/a5bb93dab2d4dd81ca7224c94cd2da67)

<br/>

## Course 2 : [Advanced Learning Algorithms](https://www.coursera.org/learn/advanced-learning-algorithms?specialization=machine-learning-introduction)

- [Week 1](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week1)
    - [Practice quiz: Neural networks intuition](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week1/Practice%20quiz%20-%20Neural%20networks%20intuition)
    - [Practice quiz: Neural network model](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week1/Practice%20quiz%20-%20Neural%20network%20model)
    - [Practice quiz: TensorFlow implementation](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week1/Practice%20quiz%20-%20TensorFlow%20implementation)
    - [Practice quiz : Neural Networks Implementation in Numpy](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week1/Practice-Quiz-Neural-Networks-Implementation-in-python)
    - [Optional Labs](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week1/optional-labs)
      - [Neurons and Layers](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C2%20-%20Advanced%20Learning%20Algorithms/week1/optional-labs/C2_W1_Lab01_Neurons_and_Layers.ipynb)
      - [Coffee Roasting](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C2%20-%20Advanced%20Learning%20Algorithms/week1/optional-labs/C2_W1_Lab02_CoffeeRoasting_TF.ipynb)
      - [Coffee Roasting Using Numpy](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C2%20-%20Advanced%20Learning%20Algorithms/week1/optional-labs/C2_W1_Lab02_CoffeeRoasting_TF.ipynb)
    - [Programming Assignment](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week1/C2W1A1)
      - [Neural Networks for Binary Classification](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C2%20-%20Advanced%20Learning%20Algorithms/week1/C2W1A1/C2_W1_Assignment.ipynb)
  

  <br/>

- [Week 2](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week2)
    - [Practice quiz : Neural Networks Training](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week2/Practice-Quiz-Neural-Network-Training)
    - [Practice quiz : Activation Functions](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week2/Practice-Quiz-Activation-Functions)
    - [Practice quiz : Multiclass Classification](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week2/Practice-quiz-Multiclass-Classification)
    - [Practice quiz : Additional Neural Networks Concepts](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week2/Practice-Quiz-Additional-Neural-Network-Concepts)
    - [Optional Labs](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week2/optional-labs)
        - [RElu](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C2%20-%20Advanced%20Learning%20Algorithms/week2/optional-labs/C2_W2_Relu.ipynb)
        - [Softmax](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C2%20-%20Advanced%20Learning%20Algorithms/week2/optional-labs/C2_W2_SoftMax.ipynb)
        - [Multiclass Classification](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C2%20-%20Advanced%20Learning%20Algorithms/week2/optional-labs/C2_W2_Multiclass_TF.ipynb)
    - [Programming Assignment](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week2/C2W2A1)
      - [Neural Networks For Handwritten Digit Recognition - Multiclass](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C2%20-%20Advanced%20Learning%20Algorithms/week2/C2W2A1/C2_W2_Assignment.ipynb)
    

<br/>

- [Week 3](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week3)
    - [Practice quiz : Advice for Applying Machine Learning](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week3/Practice-Quiz-Advice-for-applying-machine-learning)    
    - [Practice quiz : Bias and Variance](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week3/practice-quiz-bias-and-variance)
    - [Practice quiz : Machine Learning Development Process](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week3/practice-quiz-machine-learning-development-process)
    - [Programming Assignment](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week3/C2W3A1)
        - [Advice for Applied Machine Learning](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C2%20-%20Advanced%20Learning%20Algorithms/week3/C2W3A1/C2_W3_Assignment.ipynb)

<br/>


- [Week 4](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week4)
    - [Practice quiz : Decision Trees](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week4/practice-quiz-decision-trees)
    - [Practice quiz : Decision Trees Learning](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week4/practice-quiz-decision-tree-learning)
    - [Practice quiz : Decision Trees Ensembles](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week4/practice-quiz-tree-ensembles)
    - [Programming Assignment](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C2%20-%20Advanced%20Learning%20Algorithms/week4/C2W4A1)
        - [Decision Trees](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C2%20-%20Advanced%20Learning%20Algorithms/week4/C2W4A1/C2_W4_Decision_Tree_with_Markdown.ipynb)

#### [Certificate of Completion](https://coursera.org/share/488a85ebac2ffdc416c315aa1098af96)        

<br/>

## Course 3 : [Unsupervised Learning, Recommenders, Reinforcement Learning](https://www.coursera.org/learn/unsupervised-learning-recommenders-reinforcement-learning?specialization=machine-learning-introduction)

<br/>

- [Week 1](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week1)
    - [Practice quiz : Clustering](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week1/Practice%20Quiz%20-%20Clustering)
    - [Practice quiz : Anomaly Detection](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week1/Practice%20Quiz%20-%20Anomaly%20Detection)
    - [Programming Assignments](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week1/C3W1A)
        - [K means](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week1/C3W1A/C3W1A1/C3_W1_KMeans_Assignment.ipynb)
        - [Anomaly Detection](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week1/C3W1A/C3W1A2/C3_W1_Anomaly_Detection.ipynb)

<br/>

- [Week 2](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week2)
    - [Practice quiz : Collaborative Filtering](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week2/Practice%20Quiz%20-%20Collaborative%20Filtering)
    - [Practice quiz : Recommender systems implementation](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week2/Practice%20Quiz%20-%20Recommender%20systems%20implementation)
    - [Practice quiz : Content-based filtering](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week2/Practice%20Quiz%20-%20Content-based%20filtering)
    - [Programming Assignments](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week2/C3W2)
        - [Collaborative Filtering RecSys](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week2/C3W2/C3W2A1/C3_W2_Collaborative_RecSys_Assignment.ipynb)
        - [RecSys using Neural Networks](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week2/C3W2/C3W2A2/C3_W2_RecSysNN_Assignment.ipynb)

<br/>

- [Week 3](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week3)
    - [Practice quiz : Reinforcement learning introduction](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week3/Practice%20Quiz%20-%20Reinforcement%20learning%20introduction)
    - [Practice Quiz : State-action value function](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week3/Practice%20Quiz%20-%20State-action%20value%20function)
    - [Practice Quiz : Continuous state spaces](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week3/Practice%20Quiz%20-%20Continuous%20state%20spaces)
    - [Programming Assignment](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/tree/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week3/C3W3A1)
        - [Deep Q-Learning - Lunar Lander](https://github.com/greyhatguy007/Machine-Learning-Specialization-Coursera/blob/main/C3%20-%20Unsupervised%20Learning%2C%20Recommenders%2C%20Reinforcement%20Learning/week3/C3W3A1/C3_W3_A1_Assignment.ipynb)
#### [Certificate of Completion](https://coursera.org/share/9b91d610a299397aebb15990a75681fa)


### [Specialization Certificate](https://coursera.org/share/a15ac6426f90924491a542850700a759)

<br/>

<br/>

<hr/>

<div align="center">

                        
### Stargazers over time
[![Stargazers over time](https://starchart.cc/greyhatguy007/Machine-Learning-Specialization-Coursera.svg?variant=adaptive)](https://starchart.cc/greyhatguy007/Machine-Learning-Specialization-Coursera)                 

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fgreyhatguy007%2FMachine-Learning-Specialization-Coursera&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)

</div>

### Course Review :

This Course is a best place towards becoming a Machine Learning Engineer. Even if you're an expert, many algorithms are covered in depth such as decision trees which may help in further improvement of skills.

**Special thanks to [Professor Andrew Ng](https://www.andrewng.org/) for structuring and tailoring this Course.**

<br/>

<hr/>

#### An insight of what you might be able to accomplish at the end of this specialization :

* <i>Write an unsupervised learning algorithm to **Land the Lunar Lander** Using Deep Q-Learning</i>

    - The Rover was trained to land correctly on the surface, correctly between the flags as indicators after many unsuccessful attempts in learning how to do it.
    - The final landing after training the agent using appropriate parameters : 

https://user-images.githubusercontent.com/77543865/182395635-703ae199-ba79-4940-86eb-23dd90093ab3.mp4

* <i>Write an algorithm for a **Movie Recommender System**</i>
    
    - A movie database is collected based on its genre.
    - A content based filtering and collaborative filtering algorithm is trained and the movie recommender system is implemented.
    - It gives movie recommendentations based on the movie genre.

![movie_recommendation](https://user-images.githubusercontent.com/77543865/182398093-c7387754-34a9-4044-b842-0085060c3525.png)

* <i> And Much More !! </i>


Concluding, this is a course which I would recommend everyone to take. Not just because you learn many new stuffs, but also the assignments are real life examples which are *exciting to complete*. 

<br/>

**Happy Learning :))**


 
 
