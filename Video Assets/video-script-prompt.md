#Instructions:
Review the .adoc page content and when you detect that the content is eligible for video, create the video script.

#Video Eligibility criteria:
Complexity: A concept or procedure is hard to explain or visualize using text alone.
Procedure Flow: A complex procedure (how-to guide, task steps) has multiple steps or crosses different products and UIs.
Length: A topic takes many pages of text to explain, but isn't difficult to grasp.

#General Guidelines:
- Assuming that the video will run about 150 words per minute, aim for a video of 2 minutes, or a script of 400 words or less.
- The script will be read by an off-screen narrator. Think of the narrator as a friend explaining the information in the Help topic.
- Don't name the video or reference a video's name.
- Avoid repetition of words, phrases, and information.

#Style
- Write simple, declarative sentences (subject, verb, object)
- Avoid long introductory phrases.
- Use a friendly, a bit casual, and helpful tone, without being too chatty.
- Avoid using the verbs "enabling" and "ensuring," which are weak verbs. 
- Don't use permissive language ("lets the user" or "allows the user to...").
- Remove participle phrases and dangling modifiers at the ends of sentences ("Use the New button, resulting in saving your data").
- Use adverbs and adjectives very sparingly. Often the text is strong enough without modifiers.

-Introduce the topic that the video covers.
-Give 2-3 learning objectives.

In small, easy-to-understand increments, guide the viewer through the concept or task.
Write mainly in the second person by using "you" sentences that address the viewer to explain user actions or benefits.
Write in the third person sparingly by using "we" sentences only to refer to the product (example, "we're using this feature to...").
Limit tech jargon to what's necessary, and use clear, conversational language.
End the script with "To learn more, check out these resources." Don't add anything after that phrase.

#Example 1
Hi there!
Today, we're going to talk about protecting application property values in [Product Name]. This is a crucial step for ensuring the security of your application's sensitive information.
These protected application values are encrypted and stored securely.

Before you begin, make sure you have the prerequisites in place. You need:
An active platform account.
A deployed application.
ADD PROPERTIES
Now, sign into [Your Platform] and go to [Product Name].
Under Applications, click your desired application that you want to protect property values.
Then, select Manage Application.
On the Settings page, click the Properties tab.
Depending on how many properties you want to protect at once, you can:
Click Table view to protect properties one at a time or click Text View to protect multiple properties.
For example, click Table View.
In the New Key field, add a property to protect, for example: environment.
Then, in the New Value field, add the value for that property, for example: production.
Repeat the same process to add other properties.
PROTECT PROPERTIES VALUES
Now, to protect a specific property value, click Protect, and then Protect Value to confirm.
Before you proceed, choose from one of these options:
If your application has already been deployed, click Apply changes otherwise, click Deploy Application.
For our example, we'll click "Apply Changes".
And you are done!
In the Properties tab, the values for properties that you just protected are now no longer visible to you or any other user.
REPLACE A PROTECTED PROPERTY VALUE
After you protect a property value, you can't retrieve it. However, you can replace the protected property value with a new protected value.

In the Table View, click the icon next to the protected value that you want to replace.
Click the menu icon next to the value and then click Replace protected value.
Enter a new value in the field, and then click Apply, and then Apply changes.
And you are all set to protect your properties' values.
To learn more, check out these resources.


#Example 2
Hey there!
Today, we're going to talk about reviewing application versioning using [Product Name].
Application versioning allows you to view the configuration change history of the app, make updates to a deployed application, view application statuses, and roll back the configurations to earlier versions.


Before you begin, make sure you have the prerequisites in place. You need:
A platform account.
A deployed application.
VIEW CONFIGURATION CHANGE HISTORY
Now, sign into [Your Platform] and go to [Product Name].
In the navigation menu, click Applications, and select an application.
Then, select Manage Application.
On the management panel, under Settings, click the Configuration button.
The Configuration page displays the configuration history about the application.
The Config changes tab displays each change to the successfully deployed application configuration.
The All changes tab displays a history of versions created from the configuration deployment.
UPDATE A DEPLOYED APPLICATION
On the Settings page for the application, you can make these changes to a deployed configuration:
Upload a new application package.
Make changes to the deployment target and resource allocation.
Modify values for properties.
Once your changes are made, click Apply Changes.
At the top of the Settings page, a status bar appears.
VIEW APPLICATION STATUS

After you have applied the configuration changes for the application, click View Status to view the status for each replica in the deployment.

Once the configuration is successfully applied and deployed, the confirmation message appears at the top of the screen.
ROLL BACK TO A PREVIOUS CONFIGURATION
If you deploy a configuration that has issues, you can roll back to a previous successful version of the configuration.
In the Config changes history, select the version to which to roll back.
Click Apply Changes.

The configuration moves to the top of the list in the Config changes history and displays the new deployment date.
By using application versioning in [Product Name], you can maintain a clear history of your app's changes, perform smooth updates, and quickly revert to previous versions if needed. This makes your app management more efficient and reliable.
To learn more, check out these resources.
