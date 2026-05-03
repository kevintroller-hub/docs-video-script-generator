# Video Script Sample

This is a completed video script example, extracted from `video-script-samples.pdf`. Use this as the reference for tone, structure, and level of detail.

## Metadata

```
Title:              Protect Application Property Values
Short Description:  Shows how [Product Name] enables you to protect application
                    property values by displaying the property name, but not its value,
                    in [Product Name].
Technical Writer:   [Author Name]
Video destination:  Help topic, in-app, Trailhead, devdoc
Slack channel:      #[your-video-channel]

Resources:
● [Product] Documentation on Protecting Application Property Values
```

## Script

| # | Audio | Action On Screen | Comments |
|---|-------|-----------------|----------|
| 1 | Hi there! Today, we're going to talk about protecting application property values in [Product Name] through [Product Name]. This is a crucial step for ensuring the security of your application's sensitive information. These protected application values are encrypted and stored in the [platform] secrets manager. | Slide presentation | |
| 2 | Before you begin, make sure you have the prerequisites in place. You need: An active [Your Platform] account. A application. Access to [Product Name] | Slide presentation | |
| 3 | ADD PROPERTIES — Now, sign into [Your Platform] and go to [Product Name]. Under Applications, click your desired application that you want to protect property values. Then, select Manage Application. On the Settings page, click the Properties tab. Depending on how many properties you want to protect at once, you can: Click Table view to protect properties one at a time or click Text View to protect multiple properties. For example, click Table View. In the New Key field, add a property to protect, for example: environment. Then, in the New Value field, add the value for that property, for example: production. Repeat the same process to add other properties. | Shows the action to log into [Your Platform] and go to the RTM UI section. Then show the action to add properties and their values. | |
| 4 | PROTECT PROPERTIES VALUES — Now, to protect a specific property value, click Protect, and then Protect Value to confirm. Before you proceed, choose from one of these options: If your application has already been deployed, click Apply changes otherwise, click Deploy Application. For our example, we'll click "Apply Changes". And you are done! In the Properties tab, the values for properties that you just protected are now no longer visible to you or any other user. | Shows the action to protect property values. | |
| 5 | REPLACE A PROTECTED PROPERTY VALUE — After you protect a property value, you can't retrieve it. However, you can replace the protected property value with a new protected value. In the Table View, click the icon next to the protected value that you want to replace. Click the menu icon next to the value and then click Replace protected value. Enter a new value in the field, and then click Apply, and then Apply changes. And you are all set to protect your properties' values. | Show the action to replace a protected property value. | |
| 6 | Thank you for watching and see you in the next video! | Outro Salesforce Slide. | |
