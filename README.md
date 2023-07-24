FOLLOW THIS TUTORIAL: https://medium.com/slalom-technology/devops-in-salesforce-implementing-your-pipeline-ce540cb106e3

In Github the file to config your pipeline or action is located at .github/workflows/blank.yml
Feel free to edit it as you need.
Just set your repository secret variables like in BitBucket and it will work!

Feel free to follow along with us as we implement a pipeline! Before you start, please ensure you have the following:

A Salesforce Developer Edition org (this will act as your CI environment)
Salesforce CLI
Visual Studio Code (with the Salesforce extension pack)
A free Bitbucket Cloud account
Git
OpenSSL
Create a Self-Signed SSL Certificate and Private Key

Open your terminal/ command prompt. Change your working directory to your desktop, and create a new directory to store the generate files:
#!bash $ mkdir certificates $ cd certificates
Generate an RSA private key, replacing <password> with your own password
#!bash $ openssl genrsa -des3 -passout pass:<password> -out server.pass.key 2048
3. Create a key file from the server.pass.key file using the same password from the previous step
#!bash $ openssl rsa -passin pass:<password> -in server.pass.key -out server.key
Delete the server.pass.key
#!bash $ rm server.pass.key
Request and generate the certificate. You will be asked a series of questions, these can be completed or skipped. When prompted for the challenge password, press enter to skip
#!bash $ openssl req -new -key server.key -out server.csr
Generate the SSL certificate
#!bash $ openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt
Encrypt the server.key for secure storage in the repository

Generate a key and initialisation vector (iv) to encrypt your server.key locally, replacing <passphrase> with your own phrase. Make a note of the key and iv output, you will need them in the next step AND later stage.
openssl enc -aes-256-cbc -k <passphrase> -P -md sha1 -nosalt
Navigate to where your certificates created in the previous step are stored in your terminal. Encrypt your server.key using the key and iv values.
openssl enc -nosalt -aes-256-cbc -in server.key -out server.key.enc -base64 -K <key-value> -iv <iv-value>
Create a Connected App in Salesforce

Log in to the developer edition you’ll use as your CI environment
From Setup, go to App Manager and click New Connected App.
Enter the following information:
Connected App Name: Bitbucket CI
API Name: Bitbucket_CI
Contact Email: your email address
Tick Enable OAuth Settings
Callback URL: http://localhost:1717/OauthRedirect
Tick Use digital signatures
Select Choose file and upload your server.crt file
Add the following OAuth Scopes:
▹ Access and manage your data (api)
▹ Perform requests on your behalf at any time (refresh_token, offline_access)
▹ Provide access to your data via the Web (web)
Click Save
4. On your Connected App page, make a note of your Consumer Key

5. Click Manage, then Edit Policies

6. Under OAuth Policies, Set Permitted Users to: Admin approved users are pre-authorised

7. Click Save

Create a Permission Set in Salesforce

From Setup, go to Permission Sets, select New
Label: Bitbucket CI
Click Save
Select our new permission set, and click Manage Assignments, then Add Assignments
Add your user
Go back to Setup > Manage Connected Apps > Bitbucket CI
Select Manage and under the section Permission Sets: click Manage Permission Sets
Select Bitbucket CI, and hit Save
Setup your repository

Whilst logged in to Bitbucket, import a new repository using our prepared source code (make your repository private).
2. Go to Repository Settings, and click the Settings sub-menu under the Pipelines heading
Enable Pipelines
Click the Repository Variables sub-menu under the Pipelines heading
Add the key and iv values created during the section: ‘Encrypt the server.key for Secure Storage in the Repository’
Name: DECRYPTION_KEY
Value: <key-value>
Tick secured, and select Add
Name: DECRYPTION_IV
Value: <iv-value>
Tick secured, and select Add
6. Add the following variables

Name: SF_USERNAME
Value: Your CI environment Salesforce username
Select Add
Name: SF_SECRET_KEY_FOR_CLIENT_ID
Value: Consumer Key we made a note of when creating our Bitbucket CI Connected App
Tick secured, and select Add
Prepare your project

On the source page of your repository, hit the Clone button, select the HTTPS dropdown and copy the text beginning with ‘git clone…’
Open your terminal/command prompt and change your directory to where you would like to store your project
Execute the contents of step one (git clone…)
In VS Code, Open the folder created during the cloning process
Copy and paste the server.key.enc from your certificates folder to the root of your project
Use VS Code’s source control integration to stage, commit and push the certificate to your master branch (more info: here)
At this point, your commit has been pushed to your repository and Bitbucket Pipelines has been triggered. If your pipeline status is ‘Failed’, it implies there is an issue with your connected app authorisation or variables. With any luck, you’re seeing a nice green tick. Now check your Salesforce org — you should notice something new on your homepage, we just need to manually activate it:

In Salesforce, navigate to the Sales app, hit the Setup icon and select Edit Page
Click the Pages dropdown, and select Home Page Default
Hit Activation…, then Assign as Org Default, and Save
After selecting the back button and navigating back to the Sales app home page, you should be greeted with your new component congratulating you on your success
