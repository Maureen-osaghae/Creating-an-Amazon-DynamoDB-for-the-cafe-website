<h1>Working with Amazon DynamoDB</h1>
<h2>Lab overview and objectives</h2>
<p>In this lab, you use Amazon DynamoDB to store and manage menu information. Using databases, such as DynamoDB, simplifies data management because you can easily query, sort, edit, and index data. you will use both the AWS Command Line Interface (AWS CLI) and the AWS SDK for Python (Boto3) to work with DynamoDB. </p>

<p>In upcoming labs, you will use application programming interface (API) calls from the café website to dynamically retrieve and update data that's stored in a DynamoDB table. </p>
After completing this lab, you should be able to:
<ol>
      <li>Create a new DynamoDB table </li>
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

<img width="840" alt="image" src="https://github.com/user-attachments/assets/439239d3-c500-4deb-8ee2-95b14c866cb1" />

You should find one record with two attributes: product_name and product_id. Add a second record to the table.  Update the JSON data to create a new record:
<ol>
      <li>Return to the AWS Cloud9 IDE and load the not_an_existing_product.json file in the text editor.</li>
      <li>Replace the product_name value of <best cake> with best pie</li>
</ol>

<img width="959" alt="image" src="https://github.com/user-attachments/assets/76fe66e1-6780-4d6b-ab6d-cee36e3ef0e6" />

Do not change the product_id value. In the upper left, choose File > Save to save your changes.
To add the new record, run the previous command. Notice that this command is the same AWS CLI command that you used to add the first record.  Again, view the new record in the table by using the console:
<ol>
      <li>Return to the Item explorer in the DynamoDB console.</li>
      <li>Confirm that the FoodProducts table is selected.</li>
      <li>Choose Scan</li>
      <li>Choose Run</li>
      <li>Under Items returned, review the data. </li>
</ol>

<img width="959" alt="image" src="https://github.com/user-attachments/assets/3cbede96-1d2a-4935-95bf-3efc3af829af" />

Because the product_id attribute is not the primary key of the table, it doesn't need to be unique, and a new record is inserted successfully. If the value of product_name is different, a new record is created in the table. What do you think will happen if you try to insert a duplicate record? 
Return to the AWS Cloud9 IDE and try re-running the previous AWS CLI command (the up key helps here). Don't make any changes to the JSON record. 

In the DynamoDB console, choose Run again and review the Items returned list. Did you notice any changes.
💁‍♂ When a primary key doesn't exist in the table, the DynamoDb put-item command inserts a new item. However, if the primary key already exists, this command replaces the existing record with the new record, removing any previous attributes. This behavior is why you don't see a new item in the table: the record was overwritten with identical information. The primary key prevents the same product_name values from being added multiple times.

<img width="779" alt="image" src="https://github.com/user-attachments/assets/b7d89662-7c91-47d2-acb8-be1e43e231a6" />

Now, try to insert a record with an existing primary key and a different product_id value. 
Update the JSON record:
<ol>
      <li>Return to the AWS Cloud9 IDE and the not_an_existing_product.json file.</li>
      <li>Don't change the value of product_name.</li>
      <li>Replace the product_id value of <676767676767> with 3333333333</li>
      <li>In the upper left, choose File > Save to save your changes</li>
</ol>

<img width="958" alt="image" src="https://github.com/user-attachments/assets/d81092f5-a4d9-4ac7-bb15-afb53a202724" />

Run the previous AWS CLI put-item command again: View the table data in the DynamoDB item explorer by choosing Run. 

<img width="832" alt="image" src="https://github.com/user-attachments/assets/04bf2ba3-a7ba-49b1-b958-518857298799" />

<h3>Analysis:</h3> 

The product_id value of the best pie record was replaced with the new value of product_id.
However, you don't want this behaviour. You want separate operations for adding new products and for updating product attributes. 
To implement this feature, you can refine the behavior of the put-item command with condition expressions. 

You can use condition expressions to determine which item should be modified. In this case, you must prevent records from being overwritten if they already exist in the table. The <bold>attribute_not_exists()</bold> function provides this capability.
Next, test the condition expression. You try to insert another version of the record for best pie. 

Update the JSON record:
<ol>
      <li>Return to the AWS Cloud9 IDE and the not_an_existing_product.json file.</li>
      <li>Don't change the value of product_name.</li>
      <li>Replace the product_id value of <3333333333> with 2222222222</li>
</ol>

<img width="959" alt="image" src="https://github.com/user-attachments/assets/10c3d76f-6a3a-4b84-99fb-c4a0d8470a16" />

Save your changes.

       In the AWS Cloud9 terminal, run the following AWS CLI put-item command:

aws dynamodb put-item \
--table-name FoodProducts \
--item file://../resources/an_existing_product.json \
--condition-expression "attribute_not_exists(product_name)" \
--region us-east-1

The command should return this output: 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/503e46c8-09c7-4d7d-856b-af2527e1580d" />

This behavior is expected because the condition expression prevented an overwrite of the existing item. 

<h2>Task 4: Adding and modifying a single item by using the SDK</h2>

Sofía now has a good understanding of how to use the AWS CLI to control the data that is inserted into the table. She knows that the behavior for inserting data is similar with the SDK. She decides to write to the table by using Python code. 
In this task, you continue as Sofia to add and modify a single item by using the SDK.
Update the conditional_put.py script.
<ol>
      <li>In the AWS Cloud9 IDE, go to the python_3 directory.</li>
      <li>Open the conditional_put.py script.</li>
      <li>Replace the <FMI> placeholders as directed in the script. You can also refer to the code analysis in the following step.</li>
</ol>

<img width="689" alt="image" src="https://github.com/user-attachments/assets/720973ff-f59f-4224-a6bd-c3b249791a1a" />

<img width="958" alt="image" src="https://github.com/user-attachments/assets/a9905810-6cc2-4c83-ba40-922b9521bdf5" />
<img width="908" alt="image" src="https://github.com/user-attachments/assets/8d537e85-d17d-461c-bab2-717d1c6b74a2" />

In the upper left, choose File > Save to save your changes. 
Review the code to understand what it does:
<ol>
      <li>Focus on the definition of the response variable, which begins on line 49. </li>
      <li>Notice the call to the put_item operation. In the SDK for Python, the put_item operation is equivalent to the put-item command in the AWS CLI.</li>
      <li>The put_item SDK operation requires a value for Item, which defines the record that is inserted into the table.</li>
</ol>

In the AWS Cloud9 terminal, run the file. 

python3 conditional_put.py

<img width="959" alt="image" src="https://github.com/user-attachments/assets/d69ea04f-d5a5-4cc6-90d6-6ea802fa598e" />

Return to the DynamoDB item explorer, and review the updated data. You should find an entry for apple pie:

<img width="959" alt="image" src="https://github.com/user-attachments/assets/21a4799e-c319-4abf-94fc-99e682f028a4" />

In the AWS Cloud9 IDE, update the conditional_put.py script again. This time, replace the product_id value of <a444> to a555 and save the file.

<img width="731" alt="image" src="https://github.com/user-attachments/assets/f1963f0e-85b4-475a-8d02-a887223bd513" />

Run the script again: In the DynamoDB item explorer, review the table data. 
As you might expect, the item remains unchanged. Because the condition attribute_not_exists(product_name) was included in the put_item operation, the item was not overwritten. This failure behaviour is exactly the behaviour that you want.
What do you think will happen if you change only the product name (primary key) to cherry pie but keep the same attributes using that conditional expression?

In the AWS Cloud9 IDE, update the conditional_put.py script by replacing the product_name value of <apple pie> to cherry pie and saving the file. 

<img width="891" alt="image" src="https://github.com/user-attachments/assets/ee333fe1-ef3b-44b0-9ba2-43b4173bd4dc" />

Run the python3 conditional_put.py again.  In the DynamoDB item explorer, review the data: As expected, a new record was added to the table. Now, only new products will be added to the table.  This feature prevents accidental updates to existing records when more records are inserted. 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/83b4e28b-85af-4910-94af-a1ae2454e95a" />

<h2>Task 5: Adding multiple items by using the SDK and batch processing</h2>

Sofía is glad that the café staff will be able to load individual records into the table. However, she knows that it isn't scalable for café staff to load records into the database one at a time. She knows that it's more efficient to use a batch process for loading a large quantity of records at one time. 
In this task, you continue as Sofía to implement batch processing by using the SDK.
Because this batch load contains all product records, you must delete all existing records from the table before you run it.

Delete all records:
<ol>
          <li>Select the check boxes for all the table records. </li>
          <li>From the Actions menu, choose Delete item(s). </li>
          <li>In the pop-up window confirmation box, enter Delete and choose Delete items</li>
</ol>

In the AWS Cloud9 IDE, open the resources > test.json file, and review the data. This file contains six records that you use to test the batch-load script. Notice that this file contains multiple entries for apple pie on purpose.

<img width="958" alt="image" src="https://github.com/user-attachments/assets/4096be48-5872-4ba0-b9dd-f515c6355409" />

Now, update the script that performs the batch load. Update the test_batch_put.py script:
<ol>
      <li>In the AWS Cloud9 IDE, open the python_3 > test_batch_put.py script.</li>
      <li>Update the <FMI_1> placeholder with the FoodProducts table name. </li>
      <li>Replace the <FMI_2> with the product_name primary key name. </li>
      <li>In the upper left, choose File > Save to save your changes.</li>
</ol>

<img width="959" alt="image" src="https://github.com/user-attachments/assets/244a4a91-bc0c-47c5-ae84-6a5d20cbfcfc" />

To understand what the script does, review the code:
<ol>
      <li>The table that will be written to by the script is defined in the table variable on line 11. </li>
      <li>The with statement that begins on line 12 calls batch_writer(), which opens the connection to the database. </li>
      <li>Then, the code loops through each record and inserts the new data into the FoodProducts table:</li>
</ol>

table = DDB.Table('FoodProducts')
with table.batch_writer(overwrite_by_pkeys=['product_name']) as batch:
   for food in food_list:
       price_in_cents = food['price_in_cents']
       product_name = food['product_name']

In the AWS Cloud9 terminal, run the file: 
python3 test_batch_put.py

After the script completes, the terminal should show the following output: 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/70761fdc-7f68-4de7-b79f-e7f80c70c2a1" />

What is the price_in_cents value that you expect to find for apple_pie? Validate your theory by checking the data in the console. 
In the DynamoDB Item explorer, select the FoodProducts table, and run the scan again. 

<img width="706" alt="image" src="https://github.com/user-attachments/assets/be8c08cd-e3e7-4d86-9340-3d00ea2d4af2" />

Instead of keeping the first value of price_in_cents, for apple_pie, the most recent value in the data file was applied. Why did this behavior happen?
With single-item PUT requests (put_item), you can avoid overwriting duplicate records by including a condition. However, with batch inserts, you have two options for handling duplicate keys. You can either allow the overwrite, or you can cause the entire batch process to fail. 

Review the test_batch_put.py script again. Focus on line 12. The overwrite_by_pkeys=['product_name'] parameter is included in the batch_writer method. This parameter tells DynamoDB to use last write wins if the key already exists. Last write wins is why the price_in_cents attribute was updated for apple pie. 
However, you know that the café doesn't want the database to add incorrect values. For this dataset, it's better for the load to fail when duplicate product_name values are found instead of allowing the update to add incorrect values. 

you must change the script so that it fails when duplicates are included in the batch. You can then review and clean up the data. To implement this feature, you remove the overwrite_by_pkeys parameter from the batch_writer method. To prepare for the production data load, go to the browser tab with the DynamoDB console, and delete all records from the table as you did in the previous steps. 

You can fix the overwrite behavior by updating the test_batch_put.py script and preparing to load the production data. 
<ol>
      <li>In the AWS Cloud9 IDE, open python_3 > test_batch_put.py.</li>
      <li>Update line 12 by changing <with table.batch_writer(overwrite_by_pkeys=['product_name']) as batch> to the following and saving the file:</li>
</ol>

with table.batch_writer() as batch:

<img width="959" alt="image" src="https://github.com/user-attachments/assets/1801e5e8-b49a-4b6b-bc03-af840fc687e4" />

Now run the script again:
python3 test_batch_put.py

You will notice errors, which is what what you want this time. 

<img width="943" alt="image" src="https://github.com/user-attachments/assets/0a958e1d-7cc6-4701-985a-f9af01891ae5" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/289b87e9-8784-44d1-b43f-2eeb9bcdc391" />

The important feedback in this output is the ClientError: An error occurred (ValidationException) when calling the BatchWriteItem operation: Provided list of item keys contains duplicates. 
In AWS Cloud9, review the contents of the resources/website/all_products.json file. You will find many items. These items have several attributes, and some include an optional integer attribute called specials.
In order to load the raw JSON used in the website, you use a new script called batch_put.py. It is very similar to the test_batch_put.py script. This script allows for the optional integer special attribute and also maps the names of more fields to the correct DynamoDB attribute types.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/974ea682-beb7-4b54-bffe-bc1b073c2697" />

Modify the python_3/batch_put.py script. 
<ol>
      <li>Replace <FMI> with FoodProducts</li>
      <li>In the upper left, choose File > Save to save your changes.</li>
</ol>

<img width="959" alt="image" src="https://github.com/user-attachments/assets/18df0514-4e9f-4b24-b4b3-c68d39ab7cf6" />

Run the script:
python batch_put.py
After the script completes, the terminal should show the following output:

<img width="959" alt="image" src="https://github.com/user-attachments/assets/26f81311-bf46-47cf-b74d-9c923de2a6f5" />

In the DynamoDB Item explorer, review the inserted data. You should now see 26 items! 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/dcb8ba5f-9179-40c8-94f0-0a8c132711d0" />






















































