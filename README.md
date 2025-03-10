<h1>Working with Amazon DynamoDB</h1>
<h2>Lab overview and objectives</h2>
<p>In this lab, you use Amazon DynamoDB to store and manage menu information. Using databases, such as DynamoDB, simplifies data management because you can easily query, sort, edit, and index data. you will use both the AWS Command Line Interface (AWS CLI) and the AWS SDK for Python (Boto3) to work with DynamoDB. </p>

<p>In upcoming labs, you will use application programming interface (API) calls from the café website to dynamically retrieve and update data that's stored in a DynamoDB table. </p>
After completing this lab, you should be able to:
<ol>
      <li>reate a new DynamoDB table </li>
      <li>Add data to the table</li> 
      <li>Modify table items based on conditions </li>
      <li>Query the table </li>
      <li>Add a global secondary index to the table</li>
      </ol>

When you start the lab, the following resources are already created for you in the AWS account:
<ol>
      <li>AWS Cloud9 integrated development environment (IDE) instance</li>
      <li>Amazon Elastic Compute Cloud (Amazon EC2) instance for authoring code and running commands through the AWS CLI</li>
</ol>

<img width="407" alt="image" src="https://github.com/user-attachments/assets/a3747b45-6aff-4abd-8305-f339c291ffa8" />

At the end of this lab, your architecture should look like the following example:

<img width="440" alt="image" src="https://github.com/user-attachments/assets/843e32f0-415f-4de2-92f1-2d43e9fadf32" />

<h2>Business Scinerio</h2>

<p>The café website is up and running, and the café staff noticed a significant increase in new customer visits. Multiple customers also mentioned that it would be helpful if the website had an up-to-date menu. They could then use the menu to check the availability of food items before going to the café. 
Frank and Martha ask Sofía to explore whether she can implement this feature for customers. Sofía is feeling more confident in her coding skills and has also been learning about different ways to store information in AWS. She knows that before they can dynamically update data on the website, she must first choose a data storage service to hold the data. She also needs to learn how to manage table data, load the product records, and create scripts to retrieve information from the data platform.</p>

<img width="392" alt="image" src="https://github.com/user-attachments/assets/4021f6c3-126f-420e-99d5-9ed08a794c3e" />

<h2>A business request from the café: Store menu information in the cloud</h2>

Frank and Martha mentioned to Sofía that they want the website to dynamically update its menu information. To prepare for this new functionality, Sofía decides to store this information in DynamoDB. 
Café staff must be able to retrieve information from the table. Sofía decides to create one script that retrieves all inventory items from the table and another script (as a proof of concept) that uses a product name to retrieve a single record.
For this first challenge, you take on the role of Sofía. You use the AWS CLI and the SDK for Python to configure and create a DynamoDB table, load records into the table, and extract data from the table.

<h2>Task 1: Preparing the lab</h2>
Before I can start this lab, I must import some files and install some packages in the AWS Cloud9 environment that was prepared for me.
       Connect to the AWS Cloud9 IDE.
       <ol>
          <li>From the Services menu, search for and select Cloud9.</li>
          <li>In the Cloud9 Instance pane, choose Open IDE.</li>
          </ol>
          The AWS Cloud9 IDE loads in a new browser tab.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/5ca9e8bb-32b3-4da7-a7d4-f49d5ac18cf9" />

Download and extract the files  needed for this lab. Run the following command in the same terminal:
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCDEV-2-91558/03-lab-dynamo/code.zip -P /home/ec2-user/environment

<img width="959" alt="image" src="https://github.com/user-attachments/assets/2cab73e9-f4fc-4a99-b5b7-5c9359187be9" />

The code.zip file was downloaded to the AWS Cloud9 instance and is now in the left navigation pane. Extract the file by running the following command:

unzip code.zip

<img width="959" alt="image" src="https://github.com/user-attachments/assets/ccc45bee-d1c2-429b-a57d-b0c2d60f28b6" />

This action extracts the files from the code.zip. In the Environment pane on the left, is the new folder that is named resources. 

I ran a script that upgraded the version of Python installed on the Cloud9 instance. It will also upgrade the version of the AWS CLI installed.
To set permissions on the script and then run it, run the following commands:

chmod +x ./resources/setup.sh && ./resources/setup.sh

<img width="959" alt="image" src="https://github.com/user-attachments/assets/3a1400bf-b7da-458d-9523-9e6bee8b9cc2" />

To verify the AWS CLI version and also verify that the SDK for Python is installed. Confirm that the AWS CLI is now at version 2 by running the aws --version command.

<img width="933" alt="image" src="https://github.com/user-attachments/assets/57d1672c-de86-4132-80be-1eefcbd061f2" />

<img width="958" alt="image" src="https://github.com/user-attachments/assets/8c3e02e1-b4c4-4301-b5b6-f5cad5add809" />

<h2>Task 2: Creating a DynamoDB table by using the SDK for Python</h2>
To store and dynamically manage the café's website menu items, Sofía decides to create a new DynamoDB table. 
In this task, you take on the role of Sofía to create and define the new DynamoDB table.
Initially, I create this table with only one attribute. Because every DynamoDB table requires a primary key, this attribute becomes the primary key for the table. Each value used as a primary key must be unique. 
The product_name is the first attribute I define in the table. The product_name attribute works well because the café's product names should not be duplicated. Also, the café wants to use the product names to query details about each record.

First, I will verify that no tables exist in the environment by using the AWS Management Console: 
From the Services menu, search for and select DynamoDB.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/46d1cedd-0f5b-4d8f-892f-f673bb3e517c" />

To edit the script that will create the DynamoDB table:
<ol>
      <li>Return to the AWS Cloud9 IDE browser tab.</li>
      <li>In the navigation pane of the AWS Cloud9 IDE, expand the python_3 directory. </li>
      <li>Open the create_table.py script by double-clicking it.</li>
      <li>Replace the <FMI_1> placeholder with the table name, which is: FoodProducts</li>
      <li>Save my changes.</li>
</ol>

<img width="959" alt="image" src="https://github.com/user-attachments/assets/17d94930-3f9a-4089-ac3d-3aedf394176b" />

To understand what the script does, review the code:

<ol>
      <li>The line that defines the DDB variable also configures the SDK for Python resource. </li>
      <li>It sets both the AWS service and the AWS Region that the Python script will call.</li>
</ol>

DDB = boto3.resource('dynamodb', region_name='us-east-1')

The following section provides the details that describe the new table. Note the values for TableName, KeySchema, and AttributeDefinitions. The AttributeDefinitions parameter contains only one value, which is product_name.I will add more attributes later at runtime. 

params = {
       'TableName': '<FMI_1>',
       'KeySchema': [
           {'AttributeName': 'product_name', 'KeyType': 'HASH'}
       ],
       'AttributeDefinitions': [
           {'AttributeName': 'product_name', 'AttributeType': 'S'}
       ],
       'ProvisionedThroughput': {
           'ReadCapacityUnits': 1,
           'WriteCapacityUnits': 1
       }
   }

The line that defines the table variable also creates the table:
💁‍♂ Boto requires key values arguments rather than the object literal format, so you use **params to pass the parameters to the create_table operation.

table = DDB.create_table(**params)
In the AWS Cloud9 terminal window, go to the python_3 directory, and run the following code: python3 creat_table.py
This command can take several minutes to run because in the code you are waiting for the table to be ready before exiting the function. It might look like it hangs, but be patient. Once the command completes successfully, the terminal output should show the following message: 

Done
<img width="959" alt="image" src="https://github.com/user-attachments/assets/e0f1481b-8d38-41d0-adf1-6fbfb4d79e18" />

Once done (and not before), run the following command to make sure the table was successfully created: 

aws dynamodb list-tables --region us-east-1

Below is the output: 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/a338b52e-8e06-4b73-8584-f4811d28ab9e" />

Return to the browser tab with the DynamoDB console. Use the refresh icon on the far right, choose FoodProducts, and verify that the table has a created state of Active. 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/0594cc1c-2cb3-46fa-9e7f-bc284c326444" />
In the next few tasks, I will learn how to add data to the table that I created. 

<h2>Task 3: Working with DynamoDB data – Understanding DynamoDB condition expressions</h2>
Now that I have created the table, I wants to understand what happens when records are written to it. In this task, I will insert the first record into the table. 
Review the JavaScript Object Notation (JSON) data that defines the new record. In the AWS Cloud9 IDE, expand the resources folder. Open the not_an_existing_product.json file by double-clicking it.

 <img width="959" alt="image" src="https://github.com/user-attachments/assets/01df06ac-406e-4a07-87d1-665de464d6ce" />

Analysis: This file contains one item with two attributes: product_name and product_id. Both of these attributes are strings. The primary key (product_name) was defined when the DynamoDB table was created. Because DynamoDB tables are schemaless (to be exact, not bound by a fixed schema), you can add new attributes to the table when items are inserted or updated. With DynamoDB, you don't need to change the table definition before you add records that contain additional attributes. To insert the new record, run the following command. Ensure that you are still in the python_3 folder. 

aws dynamodb put-item \
--table-name FoodProducts \
--item file://../resources/not_an_existing_product.json \
--region us-east-1

Verify that the new record was added to the table by using the DynamoDB console to complete the following tasks:

<ol>
      <li>Return to the DynamoDB console and choose the FoodProducts link.</li>
      <li>Choose Explore table items.</li>
      <li>Under Items returned, review the information.</li>
</ol>

















