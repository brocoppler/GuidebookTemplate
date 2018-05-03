# Workshop Goal
Build an application from the ground up with an emphasis on developing for best practices without sacrificing performance, scalability, or overall awesomeness, whether you’re building a certification-ready integration or a custom app for you company’s unique need.
This hands-on workshop will guide you through constructing a properly secured scoped-application that accepts inbound data from both import sets and the new scripted REST API. Build a simple custom UI that allows easy real-time configuration of your app. Learn tips and Tricks with the new logging API to track usage and performance statistics. Apply debugging techniques to perform at a large scale.
# Demo App Use Case
An app to allow reporting issues in an employee parking lot.  Provide a data model that keeps track of employee’s cars and facilitates reporting issues in the lot. Someone who parks in the lot who identifies an issue (lights left on, parked in a visitor spot, parked across the lines like a jerk, etc.) can identify the car by either the license plate number or a description of the car, and report the issue via an inbound integration (scripted REST). While we want these reporting users to be able to report against cars in the database, we don’t want them to know who the driver is, for reasons of security and vandalism-prevention. A Lot Manager can see reported issues, identify affected drivers, and handle the situation without the ubiquitous company-wide email from the receptionist: “Will the driver of the silver Honda accord with license plate [SNCOD3R] please re-park their car to only take up a single spot” and ensure fast response that doesn’t necessarily require public shaming!
 
## Exercise 0.1 – Setup
This workshop starts you out with a base application with a simple data model and a test data set large enough to let you test performance.
1.	Navigate to the unique instance URL provided to you.
2.	Log on with the provided credentials.
3.	Navigate to https://github.com/joeymart/ParkingIssues and **fork** the project.
4.	From the ServiceNow main page, open **Studio** (System Applications > Studio).
5.	In the **Open Studio** module, click **“Go”** to popup the **Studio IDE**
 
6.	In the popup that appears, select **Import From Source Control:**
 
7.	Get the **HTTPS** URL from your Forked GitHub repo. It will look something like: “https://github.com/your.username/ParkingIssues.git”
8.	Paste that **HTTPS URL** into the **Import Application** popup, along with your git credentials, and click **Import**.
 
9.	After the import completes, **Select** the imported application: “Parking Issue Reporting”
 
10.	Take a moment to familiarize yourself with the existing app components
**Tip!**: To return to **Studio** at any time: 
- From the Main/Normal UI, navigate to **System Applications > Studio**
- Click the **GO** button in the **Open Studio** module
- Select **Parking Issue Reporting** from the list of Applications

## Studio
If you’re not familiar with Studio, this is your chance to use it. It’s your **IDE** for building apps in ServiceNow. You’ll use this to **create** new files, **open** existing files and **link to source control**

## Pace of this Workshop
Your goal is to learn as much as possible in this short time. If at any point you fall behind, or just want to see an example of a correct solution, you can pull from any of the branches in the GitHub repo you forked to reset to any point in the workshop. The branch names correspond to the completed exercise. For instance, the branch “ex_2_1” has all the work in exercise 1 plus exercise 2.1, ready for you to start exercise 2.2.

## What you’re starting with
  **App Name**: Parking Issue Reporting (PIR)
  **Scope**: x_pir
  **Default role**: x_pir.user
  ### Data Model
  **Car (x_pir_car)**
  General storage for make/model for cars to minimize duplication, for the 100 employees that all drive “Toyota Prius”
  make (string)
  model (string)
  display (string, read-only=true, display=true)
    **Tip! Avoid calculated fields**, especially when they dot-walk into reference fields. The “display” field is populated by a before insert/update Business Rule (“Set car display value”) which sets the read-only field any time either the **make** or the **model** field changes, rather than a calculated field which would get evaluated every time any record of the type is loaded, requiring an ad-hoc lookup to dot-walk.
**Employee Car (x_pir_employee_car)**
This table tracks employee’s cars by linking references to both the **Car** and the **Employee** and is comprised of the following fields:
car (reference to x_pir_car)
color (string)
employee (reference to sys_user)
license_plate_number (string, display=true)
 
# Exercise 1
The right data model is pivotal to any robust app. Most apps are really just a targeted data store with business logic to make dealing with that data easy for users. Getting that data store right will make your life, as the developer, and the admin’s life much easier.
Securing your data is more than just protecting potentially sensitive data, it’s building walls around your app that ensure the protection of your app’s data integrity, proper use by end users, and resiliency of your app to the crazy ways that users inevitably find to break things!
**Use Case:**
We already have a table tracking **Employee Cars**, now we need a table to store **Parking Issues** reported by our app’s users. This is a common issue tracking system that allows users to select an **Employee Car** and enter free text into a **Description** field to describe the particular issue.
## Exercise 1.1: Parking Issue Table
1.	Using **Studio**, create a new **Table** to track parking issues. 
**Label**: “Parking Issue”
**Name**: “x_pir_parking_issue” (this will get auto-populated when you enter Label)
**Columns**:
**Car** (reference to x_pir_employee_car)
**Description** (string, max_length=1000)
**Reported By** (reference to sys_user)
**Create access controls**: Checked
**User Role**: x_pir.user
**Tips!**
•	**Use singular table names**. That is, a table that stores cars is called “Car” instead of “Cars”, and this is admittedly purely subjective, but the established convention in ServiceNow is singular table names, so you’ll fit in better by following it.
•	**Don’t make users enter data you can figure out through automation**, auto-populate **“Reported By”** with the logged-in user by setting **“Default Value”** on that column to “javascript: gs.getUserID()”. This will automatically populate this field for new records. **NOTE:** Add this default value to yourexisting **Reported By** field **after** saving the table.
•	**Use auto-numbering for tables that don’t have a unique identifier.** This is on the “Controls” tab/section of the table form:
 
•	**Application Access:** Actively prepare for outside access or prevent it. If you’re not equipped to handle requests of any kind from other apps or integrations, don’t let them in. 
 
•	**Declare your table’s display column.** If anything should ever reference your table, it needs to know which column to use as the display value; otherwise you’ll get a sys_id or a timestamp, which is not a good experience. If you have a “number” or a “name” field, it will be used as the default display column if one is not already specified, so if you’ve turned on auto-numbering (see above) you’re all good!
Here’s what your table columns should look like when you’re all done:
 
 

## Exercise 1.2: Configure Table UI
Forms & Lists are the most common UI in most data-driven applications, and you get those for free in ServiceNow. Just make sure that they’re laid out logically.
Use the Form Designer to layout the new “Parking Issue” table. Make it look clean and make sense for an end user.
1.	In **Studio**, open the Form for the **Parking Issue** table:
- Select **Create New Application File** (or use the keyboard shortcut)
- Select **Form**
- Under **My tables**, select **Parking Issue [x_pir_parking_issue]**
 
2.	In the **Form Designer** UI, drag and drop the fields to make the form look pretty. 

 
3.	Click **Save**
4.	In **Normal UI**, navigate to **Parking Issue Reporting > Parking Issues** (refresh if you don’t see it)
5.	Does the column layout make sense? If not, use “Configure List Layout” to layout the columns in a way that makes sense to a user:
- Select hamburger at top-left of list
- Select **List Layout**
 
**Tip!: List Layout** or **Configure** will specify the global default. **Personalize List Columns** changes only your specific list layout. If you’re building an app, you want **List Layout (UI16)** or **Configure.**
 
 
# Exercise 1.3 Reference List Layout (sys_ref_list)
Any table that is going to be used in a reference field should have its sys_ref_list list view laid out so the reference list popup is easy to use when a user is selecting a reference
1.	In **Normal UI**, navigate to a **Parking Issues** and click **New**.
(You could also type “x_pri_parking_issue.form” into left nav search and hit enter)
2.	Click the **magnifying glass** adjacent to the **“Car”** reference field to bring up the Reference list. It should look something like below (just includes the “License Plate” field, since it’s the display field for that table)

 
3.	Select the “hamburger” icon    at the top left of the “Employee Cars” popup and select **“List Layout”** (NOT the hamburger above it on Parking Issue)
 
4.	Specify a list layout that makes it easier for the user. I’m more likely to remember “Pink Acura CL” than I am “2KNA418”. Here’s an example:
 
## Exercise 1.4: Create a new role
In our Parking Issue App, we anticipate two types of users:
•	**x_pir.user** (General users)
o	People (employees) who park in the lot that will report issues and review reported issues
•	**x_pir.lot_manager** (Admin/Configuration users)
o	People who control the behavior of the app and have visibility into its use

You already have the general user role (**x_pir.user**), let’s create a role for the lot manager
1.	Open up **Studio**
2.	Create a **Role** for lot management users. 

**Suffix:**** lot_manager
**Name:** will auto-populate to “x_pir.lot_manager”, this is your actual role name
**Description:** <Give it a description that makes sense based the role’s purpose>
**Elevated privilege:** <Unchecked/false> (you’ll see what this means in the next exercise)
**Assignable By:** <Leave this blank>
**Example:**
 

 
## Exercise 1.5: Secure with ACLs
Now that you have tables with an easy-to-use UI, and you have roles to represent your specific users, it’s time to add security to make sure the tables are used correctly, and per your design.
Those two types of users will have the following access rules:
•	**x_pir.user** (General users) allowed to:
o	Create new **Parking Issues**
o	Read/Write only **Parking Issues** that they create
o	Read the **Employee Car** table (so they can report issues against it)
	…but can’t see the car’s owner/driver (**Employee**)!
•	**x_pir.lot_manager** (Admin/Configuration users) allowed to:
o	Read/write/delete all **Parking Issues**
o	Read all **Employee Car** records (including the owner/driver)
•	Everyone else (People who are logged into the instance, but don’t have either of the specified app roles)
o	Allowed to Read **Parking Issues** conditionally, if a specified property is set
	x_pir.parking_issues_public

Access Control Rules (ACLs) for tables are defined at either the table- or field- level. A table-level rule controls access to a row, if it evaluates to false for a particular user, that user won’t be able to see the row in question. A field-level ACL controls access to a particular field. Once a user has been granted access to a row, the field-level ACLs define which fields that are allowed to read or write.
**Help! Access Control changes require elevation to security_admin.** If you don’t see “Access Control” in Studio when you try to “Create Application File”, you need to elevate using steps that follow.

1.	Elevate to **security_admin:** in **Normal UI**, click profile name at top right, and select **“Elevate Roles”**
 
2.	In the “Elevate Roles” popup that appears, check the box adjacent to **“security_admin”** and click **OK**.
 
3.	Return to **Studio**, and create a new **Access Control**
**Don’t see Access Control?** If you’ve elevated to **security_admin**, Refresh Studio
 
# ACL Overview
If you’re already an Access Control (ACL) expert, or you’re feeling independent and adventurous, skip this section. Otherwise, check out this overview of the Access Control form and then use the details here to create the set of ACLs in the exercise that follows.
 
**Type:** We’re creating ACLs for a table, so we want record here (this is most common), but you can control access to a number of different things (processor, ui_page)
**Operation:** For tables/records, we can create ACLs to target each of the CRUD operations (create/read/write/delete). 
**Admin Overrides:** Check this box if you want to exclude users with the admin role from the rule (effectively returning true
**Application:** We’re creating ACLs for our “Parking Issue Reporting” app, so that should be here. If you were modifying an ACL for your “Llama Farm” app, you’d see that here.
**Active:** True to enforce the rule, false to ignore it
**Advanced:** Write arbitrary server-side script to define whether the rule returns true (grants access) or false (rejects access). **Avoid Advanced/Scripted ACLs wherever possible.**
**Name:**
**First Choice List:** Since we’re creating a record/table ACL, select the table name that you’re controlling access to.
**Second Choice List:** This determines whether this record ACL is table-level or field-level. Simply put, it you select a field here (or “\*” to apply to all fields on the table), it’s a field-level ACL. If you leave “--None--“, it’s a table/record-level ACL
	**Description:** Good developers use this field to explain in terms a CIO can understand exactly what this rule is for. This is like adding comments to your source code. Assume that the ACLs you write will be inherited by a psychopath. Don’t make him angry.

 
**Requires Role**:  This **embedded list** is where you specify the **role(s)** required to **grant** access to the specified resource (table/field/ui_page/etc.). 
**Condition:** This is the same filter builder that you use when filtering any list of records in ServiceNow. This defines the set of rules to which the ACL applies, or put a different way, defines the set of records that you’re granting access to for the specified role.
There are three things to think about here:
1.	**Resource:** The thing you’re securing (a record or field, in our example)
2.	**Condition:** The set of records you are granting access to (if you’re dealing with a subset of records)
3.	**Script:** The thing that makes your app slow and complicated, but is sometimes necessary to represent more complex security models. You write arbitrary server-side javascript which returns true (to grant) or false (to reject). **Avoid This.** Sometimes it’s necessary, though.
 
## Exercise 1.5 (continued) Secure with ACLs
Ok, you’re either already an ACL expert or you’ve gone through the ACL overview and you’re ready to start creating some Access Control Rules (ACLs).
1.	Open up **Studio**, adjust **Access Controls** to meet the criteria in the table the follows (if you don’t see that option, go back and **Elevate to security_admin** and then refresh **Studio**)
If you checked the **Create ACLs** box when you created the table, you might already have an ACL to start from, just look in the **Application Explorer** in the left-hand side of **Studio** for: 
Access Control > Access Controls > **x_pir_parking_issue (create)**
Role	Table[.field]	Operation	Condition
x_pir.user	Parking Issue	Create	
x_pir.user	Parking Issue	Read / Write	Reported By is Logged In User
x_pir.user	Employee Car	Read	
x_pir.user	Employee Car[.Employee]	Read	Employee is Logged in User
x_pir.user	Employee Car	Create	
x_pir.lot_manager	Parking Issue	Read / Write / Delete	
x_pir.lot_manager	Employee Car	Read	
Any logged-in user	Parking Issue	Read	Property “x_pir.parking_issues_public” is “true” (Advanced: requires script)
**Tip!:** Want to **test your ACLs while** you’re building them? Open up an **Incognito Tab** or a session in a **Different Browser** and **impersonate** a user with the role being tested (you can create a new user and assign them the role)
**Tip!:** Unsure what your **filter Condition** should be? **Open a list** of records for the target table, and build your filter there, then you can actually see which rows will apply.

**Hint:** The **Reported By** field on the **Parking Issue** table stores the user that created the issue, and you can retrieve the logged-in user via the function call **“javascript: gs.getUserID()”**
 
**Tip!: Avoid Scripts in ACLs for ideal performance.** Scripts are evaluated after data has been queried, once for each row in the result set. These microseconds can add up and result in slow page loads.
**Tip!:** You may have noticed that when you created a field-level ACL on x_pir_employee_car.employee, the system automatically created an ACL on x_pir_employee_car.*. Since the system defaults to deny, this “*” rule grants access to all fields on the table, so that your rule targeting just the “employee” field supersedes that rule just for that field.
 
## Exercise #1.6 [STRETCH!]: Client-Side Scripting
This is a stretch goal. You don’t need to do it, and you probably won’t have time to complete it, but we encourage you to try if you have time during this workshop or later. 
Our parking lot has installed charging stations for electric cars right up front, but on days when our lot fills up, some people break the rules and park their non-electric cars in the electric charging spots. Users are getting frustrated when these spots are taken by cars that don’t look like electric cars, but sometimes in their frustration, they don’t realize that a car actually is electric. Let’s help them!
1.	Create a **true/false** field on the **x_pir_employee_car** table: “supports_charging”
2.	Set that field to **true** for “Tesla Roadster” and any other **electric cars** you see in the list.
3.	Create a client script on the **Parking Issue** table that runs whenever the **Car** reference field **changes**, which adds a little power icon to the form next to the field

**Tip!:** Use provided APIs over direct DOM manipulation to ensure your app is supported, even if the underlying UI changes.
**Tip!:** Make judicious use of callbacks for round-trips to the server. Callbacks make this round-trip asynchronous, which means it won’t lock up the UI.
There are three provided client-side APIs here that you can use to solve this problem:
o	**g_form.getReference(fieldName, callback)**
	Returns the GlideRecord for a specified field.
	fieldName is the populated reference field for which to retrieve the full GlideRecord
	callback is a callback function that will get called when the round-trip to retrieve the record is complete
	The callback function accepts a single GlideRecord argument
o	**g_form.addDecoration(fieldName, icon, title)**
	Adds an icon on a fields label
	fieldName is the field to which we want to add an icon
	icon is the name of the icon (we want icon-power)
	title is the text that appears when the user mouses over the icon
o	**g_form.removeDecoration(fieldName, icon, title)**
	The antithesis of g_form.addDecoration. Remove an added decoration
	Pass same args as the addDecoration call to remove it

 
# Exercise #2: Import (+Scripting) 
Once you have a properly-secured data model, you pretty much already have a working app, since ServiceNow gives you forms, lists, and a default table API for REST/SOAP web services, but as a best practice, if you’re expecting data to be imported into your app from some external source, you should create a well-defined, properly-controlled REST endpoint and avoid using the default table API. In fact, if you want to certify your app, you’re prohibited from using direct table APIs.
This exercise will guide defining a well-defined import APIs, using Scripted REST and Import Sets for two distinct use cases
## Use Case
We want our app to accept inbound data via a defined scripted REST endpoint, as that’s a common integration standard that will allow our company’s other systems to easily integrate with our Parking Issues app by creating issues via a simple request into ServiceNow.
Additionally, we need to keep the Employee Cars data up-to-date with new employees (and new cars), via scheduled imports from HR using Import Sets and Transform Maps to accept data in one data model, and load it into our app’s different data model.
## Exercise 2.1: Scripted REST API for users to report Parking Issues
1.	Open **Studio** and Create a new **Scripted REST API**
2.	Fill out the new record for the Scripted REST Service
**Name:** Parking Issue
**API ID:** parking_issue
**Protection Policy:** Read-only

**Tip!:**  Protection Policy is mostly relevant if you’re building an app that will be distributed (e.g. by the ServiceNow Store). If you want to keep your Intellectual Property a secret, you can set this to Protected, and installations of your app will encrypt the script. If you’re not concerned about code secrecy, but you don’t want anyone changing your source code here, set it to Read-only. As a best practice, if you’re not actively prepared for unexpected changes to this API, you should protect against it by at least making this Read-only.
3.	Submit the record and explore the three new form sections that appear after reload. They each have contextual descriptions on the form that explain what they’re for. In the interest of time, we’re skipping these, but here’s general best practice for each:

**Security:** You can (and should) create an ACL (operation=REST_endpoint) to control access to your REST API

**Content Negotiation:** This is a REST thing. The default should be sufficient. If you need to accept input or return a response that’s in a weird format, you’d specify that here. JSON, XML, or text should be sufficient for most APIs.

**Documentation:** If you have documentation (which you should), link to it here. At the very least, populate the “Short Description” with enough information to give an indication of the purpose and high-level behavior of your API

Here’s what your form will look like at this point:
 
4.	Scroll down to the **Query Parameters** related list, and create one record for each of the input parameters, along with an example value (these are going to be the inputs to the API we create next!):
**color** (example value: Red)
**make** (example value: Toyota)
**model** (example value: Camry)
**license_plate** (example value: 1ABC123)
**description** (example value: taking up two parking spaces)

Here’s what the **color Query Parameter** will look like, as an example:

 
And here’s what the related list will look like:
 
**Tip!:** Use the Is Required flag to specify mandatory parameters. In our case, nothing is mandatory, we just raise an exception in the case that we didn’t have enough information

## Exercise #2.2 Scripted REST Resource
1.	The **Scripted REST Service** we’ve just created is just a wrapper for **Resources**. A Resource is where all the logic is defined. Think of it as an API Method definition. 
Find the Resources related list on the **Scripted REST Service** and click **New.**
2.	Fill out the form to define the **Scripted REST Resource:**
**Name:** report_issue
**HTTP Method:** POST. We’re creating an endpoint that’s accepting data that will be pushed to it. If we were creating an endpoint that accepted a query and returned data, we’d want GET.
**Relative Path:** Leave this as “/”. You’d use this if you wanted to support templatized path parameters, where the URL itself becomes part of the query.
**Protection Policy:** Read-only. (As with the Scripted REST Service)
**Script:** Leave this as the default for now, you’ll write this in a moment.
**Requires Authentication:** <Checked>. We don’t want this API to be publicly accessible to anyone with an internet connection.
**Requires ACL Authorization:** <Unchecked>. You can create a REST_Service ACL to control who can actually call this API. As a best practice, you should use GlideRecordSecure in the script to enforce record ACLs.
**Documentation:** Fill this out just like you would add comments to source code
3.	Save/Submit the form, then scroll down to the Related List Query Parameter Associations on the Scripted REST Resource you just created.
4.	Create a **Query Parameter Association** entry for each of the Query Parameters you created (color, make, model, license_plate, description)
 
5.	Write the **Script** for this **Scripted REST Resource.**
**inputs:** color, make, model, license_plate, description
**output:** Parking Issue number

Your script will need to do the following:
•	Identify **Employee Car (x_pir_employee_car)** record based on the input (in the request).
•	If record NOT found, return 404 and don’t create a new Parking Issue record
•	If you found the car record, create a new Parking Issue record with that car and specified description
•	Return status code 200 with newly created parking issue number in a JSON payload in the response body.

**Tip!: Avoid using GlideRecord** in the script, use **GlideRecordSecure** instead of **GlideRecord** to enforce the same record ACLs that apply when users use the UI for a consistent  and secured experience.

**Tip!:** Avoid having **too much logic** in the REST resource script. Move logic into object-oriented Script Includes. This makes higher-level logic easier to follow and increases your apps supportability.
	**Tip!:** Use REST API Explorer to test your API as you go. 

**Help! 
There’s a sample REST Resource script on the next page you can start from if you’re short on time or having trouble figuring out where to start!**
 
Here’s an example script you can start with. You’ll still need to follow the TODOs and create the referenced Script Includes as part of your app, and code the functions:

**Copy-paste this script from either of the following URLs:**
https://gist.github.com/joeymart/2c9246de1c7b303eb3526cada069e6ea
http://tinyurl.com/hx2gea2
(function process(/*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {
	// Get the query parameters from the request (all strings)
	var licensePlate = request.queryParams.license_plate+'';
	var color = request.queryParams.color+'';
	var make = request.queryParams.make+'';
	var model = request.queryParams.model+'';
	var description = request.queryParams.description+'';
	
	// Lookup the employee car using the parameters
	// TODO: Create EmployeeCar script include
	// TODO: Add method find() which accepts the specified string parms
	//       and returns a GlideRecord (if found) or null (not found)
	var car = new EmployeeCar().find(licensePlate, color, make, model);
	if (!car) {
		// Return a 404 not found
		return new sn_ws_err.NotFoundError("Car could not be found");
	}
	
	// TODO: create ParkingIssue script include
	// TODO: create create() method that accepts the car (GlideRecord)
	//       and the description (string) and returns the GlideRecord
	//       for the created Parking Issue record, or null;
	var issue = new ParkingIssue().create(car, description);
	if (!issue) {
		var myError = new sn_ws_err.ServiceError();
		myError.setStatus(500); // Internal server error
		myError.setMessage("Parking issue could not be created");
		return myError;
	}
	
	// Set response to HTTP Status 200 (OK) and return the newly created number in a JSON payload
	response.setStatus(200);
	response.setBody({"issue_number": issue.number});
})(request, response);
6.	Fire up the REST API Explorer to test this API by clicking the form link “Explore REST API” at the bottom of the form (or from Left nav, go to System Web Services > REST > REST API Explorer)
7.	In the top-left, make sure the target namespace/API point to the API we just created:
 
8.	In the **REST API Explorer** you can easily craft sample requests to consume the API you’ve just created. Let’s test a use case, where Larry Burke left on the headlights on his Blue Mitsubishi Eclipse with license plate 8ZUS565
**license_plate:** 8ZUS565
**description:** Left headlights on
leave the other options blank, if you have license_plate, you don’t need make/model/color

9.	Click **Send** at the bottom of the page

Note: You might get a warning message like the following, which is just telling you that your test might actually change data, which is ok in this case
 
10.	Look at the **Response** Status Code and Body to see if you were successful. A status code of **200** and a body that includes the newly created **Parking Issue** record number means success. Anything else means it’s debug time!
Here’s an example of a failure response which tells us that we forgot to create the Script Include “EmployeeCar”:
 
11.	Repeat the previous step until you’ve got success (i.e. a correctly created Parking Issue record). 

**Tip!: Having trouble getting things to work?** Peek ahead to **Exercise #3.2** and add some **debug logging** to help trace our script behavior
12.	Now, try another use case, where you don’t have the license plate number, but you do have color/make/model. Your script should be able to still be able to find the record:
 
13.	Again, repeat until you get success. Add debug logging per Exercise #3.2 to help.

## Exercise 2.3 [STRETCH]: Configurable Behavior
This is another STRETCH goal. If you have time, give it a try. If not, move on to the next exercise.
1.	Create a **System Property** “x_pir.issue.create_car_if_not_found”
2.	Modify your **Car Identification** script (e.g. the **EmployeeCar** script include in the provided sample, or in the **Scripted REST Resource** directly) to check this property if the car is NOT FOUND. If the property is true, create a new record in the **Employee Car** table and create the Parking Issue against it. 
**Hint:** You’ll need to lookup the make and model in the Car table (x_pir_car) and create a record there only if one does not already exist.
 
# Exercise #3 Logging & Debugging
Whether the REST API you created in exercise #2 is working perfectly or failing miserably, you’re ready to proceed to this next step.
Code written in ServiceNow doesn’t give you the luxury of breakpoints to step through server-side code like Scripted REST APIs, and even if it did, that doesn’t help diagnose some production failure that can’t be replicated. Because of this, it’s essential to add logging to custom code that can help identify behavior and help track usage, but doing so without being too cumbersome or adversely affecting behavior can be tricky.
In this next exercise, you’re going to add logging to all your custom scripts (Scripted REST and Script Includes) to help with debugging and usage tracking, but first let’s overview some of the lesser known features of the NEW logging API!

## The new Logging API
Logging API globally accessible (via "gs") from within scoped apps with built-in property-based controls by app for:
•	Configuring log verbosity (error, warn, info, debug) via property APP_SCOPE_PREFIX.logging.verbosity
•	Configuring log destination (file system, DB) via property APP_SCOPE_PREFIX.logging.destination
•	Turning session debugging on/off (logs at debug level verbosity)

**Verbosity**
There are 4 levels of verbosity, in order from least verbose to most verbose, they are: 
•	error (gs.error)
•	warn (gs.warn)
•	info (gs.info)
•	debug (gs.debug)
...this means that setting log-level for a particular app to "info" will give you info+warn+error, but NOT debug. 
...Session Debugging verbosity is "debug", so any level of log verbosity will be included in Session debug output, for JUST the app being debugged IN the session with session debugging activated.

**Destination**
Log destination of "db" will route log messages within desired logging verbosity to the table "syslog_app_scope". This new table has added fields for the App/Scope issuing the log as well as a Source Script field that links to the script that called into the logging API (when we can figure it out, currently just for Business Rules and Script Includes). When Destination is set to "db", logs will ALSO go to file system.
Log destination of "file" will log to the node's file system log without broadcasting to log listeners (except in the case of Scripts-Background)

**Usage Examples**
Consumers of this API (from JavaScript) should be able to just do:
•	gs.debug(message [, parameters]) -emit log message at debug level verbosity
•	gs.info(message [, parameters]) -emit log message at info level verbosity
•	gs.warn(message [, parameters]) -emit log message at warn level verbosity
•	gs.error(message [, parameters]) -emit log message at error level verbosity
•	parameters are optional, can be comma-delimited or a single javascript array
•	gs.isDebugging() -Returns True when EITHER session debugging is active OR log verbosity is set to debug for current app
•	gs.enableSessionScopeDebugging([app/scope name]) -Turns on session debugging for the specified app scope (or, if parameter omitted, turns on session debugging for the current app scope
•	gs.disableSessionScopeDebugging([app/scope name]) -Turns OFF session debugging for the specified app scope (or, if parameter omitted, turns off session debugging for the current app scope

**MessageFormat Placeholders**
This logging API supports the java MessageFormat placeholder replacement pattern
Currently there is support for up to 5 "varargs" placeholder arguments, any more than 5 need to be specified as a single javascript array argument
All of these are legal calls:
•	gs.info("Here's a log message from me"); // no params
•	gs.info("Here's a log message from {0}", myName); // single non array param
•	gs.info("Here's a log message from {0}.{1}", myFirstName, myLastName); // multiple "varargs" params (up to 5!)
•	gs.info("Here's a log message from {0}.{1}", [myFirstName, myLastName]); // array of n-number of params (no upper bounds)

**Exceptions**
Also support passing an Exception as the first parameter after the string message (in which case all other following parameters would be ignored).
This will automatically append the exception and stack trace to the specified message. Like so:
 		try {
 			foo.somethingThatThrows();
 		} catch (ex) {
 			gs.error("There was an error doing something", ex);
 		}

 
## Exercise #3.1 Properties Page
You can create a properties page, which will show all your app’s custom properties and allow easy adjustment to help with tuning your application. Even if you didn’t create any custom properties as part of your app, you’ll want a properties page for the 2 logging properties, which control the verbosity and destination of, your app’s log statements.
1.	Open **Studio**
2.	Create a new **System Property** for the log verbosity property
**Suffix:** logging.verbosity
**Name**: will autopopulate to include prefix (x_pir.logging.verbosity)
**Description:** Control the logging verbosity of the Parking app (Default=info)
**Choices:** off,error,warn,info,debug
**Type:** Choice list
**Value:** info
**Ignore Cache:** <Checked>

**Tip!:** You should probably always check the Ignore Cache checkbox for your custom properties. With that, the system SKIPS a global cache flush that can temporarily degrade performance.
3.	Create a new **System Property**** for the log destination property
**Suffix:** logging.destination
**Name**: will autopopulate to include prefix (x_pir.logging.destination)
**Description:** Control the logging destination of the Parking app (Default=db)
**Choices**: none,file,db
**Type:** Choice list
**Value:** db
**Ignore Cache:** <Checked>
4.	Create a new System Property Category for your app
**Name:** Parking
5.	Open the System Property Category you just created, scroll down to the related list Properties and click the **Edit** button
6.	Type the app scope (x_pir) into the Search box and the two properties you created will appear. Select them (any others you may have created for STRETCH goals) and move them to the box on the right, then click **Save**
 
7.	From **Studio,** Create a new Module
**Title:** Properties
**Application Menu:** Parking Issue Reporting
**Order:** 9999 (position it at the bottom of the list for this application)
**Link Type:** URL (from Arguments:)
**Roles:** x_pir.lot_manager,admin (you don’t want general users changing properties)
**Arguments:** system_properties_ui.do?sysparm_category=Parking&sysparm_title=Parking%20Properties
8.	Save the Module
9.	Return to the **Normal UI**, refresh the left nav, and find your **Parking > Properties** module
10.	Observe your new properties page! Any properties you associate with the Parking Property Category will automatically show up here:

 
11.	Use this property page the change the **Logging Verbosity** property to **debug**.
 
## Exercise #3.2 Add logging to Custom Script
1.	Navigate to the **Scripted REST Resource** created in Exercise #2.
2.	Add a call to “gs.info” at the top of the function to output the logged-in user’s name, and the uri they called, something like:
> gs.info(gs.getUserName() + " called " + request.uri)
3.	Add a call to “gs.debug” the result.queryParams JSON Object.
**Tip!** The Helsinki release introduces EcmaScript5 support! You can use the new native JSON.stringify method to get a serialized string representation of the JSON Object that result.queryParams returns:
> gs.debug(JSON.stringify(request.queryParams));
4.	Add a call to “gs.debug” the input parameters, try the MessageFormat log style to achieve log messages with parameters spliced into the string without the need for string concatenation:
> gs.debug("licensePlate={0}, color={1}, make={2}, model={3}",
			licensePlate, color, make, model);
5.	In the case where the car is not found after the first lookup, log a warning using gs.warn.
6.	Use **REST API Explorer** to trigger your API.
7.	In the normal UI, navigate to the **Application Logs** table (System Logs > Application Logs).
8.	Add the **Sequence** field to the list layout, and sort by it descending to see most recent entries at the top of the list (your debug message will not be there).
9.	Submit a couple of request and see the added log entries.
 
## Exercise #3.3 Session Debugging
Session debugging adds debug log output for your session, and emits it to your browser on page loads when active. This can be invaluable in debugging.
1.	In the **Normal UI**, navigate to System Applications > Applications
2.	Open the **Parking Issues Reporting** app by clicking the name (NOT the Edit button)
3.	Click the Form Link at the bottom of the page **Enable Session Debugging**.
This activates debug logging for this app for just your session. No other sessions will see the debug output
4.	Return to **REST API Explorer**, submit a **Parking Issue** 
5.	In a new tab/window, navigate to <your instance>/ui_page.do
**Tip!**: ui_page.do is just an empty page, but since Session Debug output emits the logs for the Previous Transaction in addition to the current one, you can use it to see session debug output for a transaction that doesn’t emit session debug output (like REST
6.	Observe your log messages are emitted like the following.
 
7.	In the **Normal UI** execute System Diagnostics > Session Debug > **Debug SQL (Detailed)**
This will log all SQL statements executed for each transaction in your session (irrespective of app) in addition to your app’s debug logging.
 
8.	Repeat steps to submit a **Parking Issue** via the **REST API Explorer**
9.	Reload **/ui_page.do** to get **SQL Debug** output, and scan through to see the actual SQL executed by the GlideRecord scripts you wrote! Here’s an example
[1] 13:43:11.758: [DEBUG] x_pir: licensePlate=, color=Teal, make=GMC, model=Envoy, description=lights on
[2] 13:43:11.769: Time: 0:00:00.004 for: helsinki_cc16[glide.2] /*...*/ SELECT ... FROM ((x_pir_employee_car x_pir_employee_car0 LEFT JOIN x_pir_car x_pir_car1 ON x_pir_employee_car0.`car` = x_pir_car1.`sys_id` ) LEFT JOIN x_pir_car_manufacturer x_pir_car_manufacturer2 ON x_pir_car1.`make` = x_pir_car_manufacturer2.`sys_id` ) WHERE x_pir_employee_car0.`color` = 'Teal' AND x_pir_car_manufacturer2.`name` = 'GMC' AND x_pir_car1.`model` = 'Envoy'
[3] 13:43:11.770: [DEBUG] x_pir: Create a new Parking Issue record
[4] 13:43:11.775: Time: 0:00:00.000 for: helsinki_cc16[glide.8] /*...*/ SELECT ... FROM sys_number_counter sys_number_counter0 WHERE sys_number_counter0.`table` = 'x_pir_parking_issue'
[5] 13:43:11.777: Time: 0:00:00.000 for: helsinki_cc16[glide.9] /*...*/ UPDATE sys_number_counter SET `number` = 1016 WHERE sys_number_counter.`sys_id` = '480d597a673212008db1bcb532415a29' AND (sys_number_counter.`number` = 1015 OR sys_number_counter.`number` IS NULL )
[6] 13:43:11.779: Time: 0:00:00.000 for: helsinki_cc16[glide.10] /*...*/ INSERT INTO x_pir_parking_issue (`sys_id`, `number`, `sys_updated_by`, `car`, `sys_created_on`, `sys_mod_count`, `description`, `reported_by`, `sys_updated_on`, `sys_created_by`) VALUES('6130ba17677212008db1bcb532415a68', 'PAR0001016', 'admin', 'aa1db0b6673212008db1bcb532415ae9', '2016-04-20 21:43:11', 0, 'lights on', '6816f79cc0a8016401c5a33be04be441', '2016-04-20 21:43:11', 'admin')
Lines [1] and [3] are gs.debug statements written as part of our app.
Line [2] shows the query executed to look up the employee car based on REST API inputs
Lines [4] and [5] show the SQL executed as part of auto-numbering in the parking issue table
Line [6] shows the INSERT statement to create a new parking issue table

 
## Exercise #3.4 Database Indexes
Now that you have visibility into the actual SQL statements your app is running and how long they take, you can create database indexes to cover common queries to keep performance optimal even as the table sizes grow. 
1.	In **Studio**, open the **Employee Car Table**
2.	Scroll down to the Related List **Database Indexes**. Here you can see the indexes that already exist on this table. By default, you’ll only have a PRIMARY key on sys_id, and secondary indexes for each reference field (added automatically to help with JOINs)
 
3.	We know we’re going to be querying by **license_plate_number** from our REST API, so it makes sense to create a Database Index for it. Click the **New** button on the **Database Indexes** related list.
4.	In the form that pops up, select the **License Plate Number** field, and check the Unique box:
 
5.	Click **Create Index**.
This will schedule index creation via a Run Once scheduled job that should run within a minute or so.
6.	Refresh the **Database Indexes** related list and observe the new index on license_plate_number.

**Tip!**: Database indexes you create like this get recorded as metadata as part of your app so you actually get shipped as part of your deployment (or in an update set).

# PERFORMANCE DEBUGGING

**Setup:**

We will be switching applications for this section and downloading a modified Parking Issues application version. This will include new functionality that will help us employ debugging.

The new application is ready for download. You will need to switch your repository configuration.

To do this we will need to delete the current application and import a new one

1. In the normal UI go to System Applications > Applications

2. Select the Parking Issues application

3. In the upper right corner select delete. Type the word delete and submit.

4. Once the application is removed we will import the modified version via the same steps.

a. From the normal UI open Studio

b. Select import from source control

c. Enter the new application repository:

https://github.com/cjimeneznow/ParkingIssues

Note: Credentials are not needed

5. Once the application is imported switch branches to performance-debugging_3.5-6.

Once the new application has been installed proceed to the next sections.

## Exercise #3.5 Car Location UI Action

Now that we have the new application with some added functionality we will execute the added changes and verify if we can determine if it’s working well and performing well.

We’ve added a new UI action that we can test to see how it works.

1. In the Normal UI navigate to the Parking Issues Module. In the list view click “new”.

Inline-style:

![alt text](https://github.com/cjimeneznow/parkingissuesimages/blob/master/img_3.5.1.png)

2. Use the lookup select box to select an existing car that has an employee. Use the filter conditions to select the condition where employee is not empty.

Inline-style:

![alt text](https://github.com/cjimeneznow/parkingissuesimages/blob/master/img_3.5.2.1.png)

Inline-style:

![alt text](https://github.com/cjimeneznow/parkingissuesimages/blob/master/img_3.5.2.2.png)

3. Open the new record and find the added ui action form button. Click on the car location UI Action. The UI Action will take a few seconds to retrieve the location of the user based on the employee reference field. This is a reference look up and should be relatively quick for a single record.

a. Let’s debug the performance to see where time is spent processing this request.

4. In the Normal UI click into the filter navigator and enter “Debug”. Typically selecting the debug log, debug sql detailed, and debug business rules is a good way to profile behavior and begin debugging. For now that should suffice, go ahead and enable these three modules.

Inline-style:

![alt text](https://github.com/cjimeneznow/parkingissuesimages/blob/master/img_3.5.4.png)

5. With debugging enabled let’s trigger the UI Action once more.

a. Open the same record that was created and click on “Check car location”

b. Note the session debugging output. It will look similar to the following:

Inline-style: ![alt text](https://github.com/cjimeneznow/parkingissuesimages/blob/master/img_3.5.5.png)

6. Let’s interpret the output. Typically, when debugging we want to focus on identifying any errors or unexpected behavior. Additionally, we want to look for any business rules that may be recursive, excessive or slow queries, or any timing that shows an excessive time spent on any given request or query. The output will also include ‘receding lines from previous transactions which can provide visibility into what asynchronous requests occur.

a. Does the output look as you would expect?

b. Are there any errors, repetitive queries, or slow queries that we could focus on improving?

c. Any recursive business rules?

7. A very useful resource out of the session debugging is the ability to see a stack trace for the queries that are running. This can help you determine where a query may be coming from, for example if a business rule is involved. Below is an example output:

https://github.com/cjimeneznow/parkingissuesimages/blob/master/img_3.5.7.png

## Exercise #3.6 Car manufacturer UI Action debugging

Let’s expand on the previous exercise and take a look at a more advanced UI Action execution.

1. With the same debugging enabled go to the Normal UI and click on the Parking Issues > Cars application. This will open the x_pir_car table in a list view.

2. Open any car record and notice the UI Action “Get Manufacturer Details”.

Click on the UI action. Notice there is a new attachment on the instance. This attachment is the car manufacturer details obtained via an external source.

a. Look at the debugging output. Do you see where we obtain the external resource data? You should see a debug log output that shows the source of the attachment data.

Go ahead and click on the UI action once more. Note that there is a message that appears. There is an added check on the attachment to see if it’s changed. If the attachment is the same, it will not be attached again.

3. After clicking on the UI action a second time we should profile the functionality and review if there are any issues with the execution.

a. Does the output look as you would expect?

b. Are there any errors, repetitive queries, or slow queries that we could focus on improving?

c. Any recursive business rules?

## Exercise #3.7 Employee Car Registration – List and record producer

We’ve debugged UI actions that lead to executing multiple scripts and added functionality. For the final exercise, we will look at a different work flow. We will need to switch branches in studio to acquire some changes. Please switch to branch to performance-debugging_3.7.

Once you’ve switched branches we have changed the functionality of the x_pir_employee_car table. We will want to inspect the behavior and debug for any issues. The following exercise will be all in the Normal UI.

1. Let’s incorporate other debugging options to explore other debugging capabilities. Along with the 3 modules we enabled earlier, debug log, debug sql, detailed, and debug business rules, we can see how the functionality a user will have if they have the x_pir_user role as an example.

a. If you do not have a user with this role as admin add the role to Abel Tuter and then impersonate them.

2. Go ahead and load the /x_pir_employee_car_list.do list.

3. From the list view notice that the employee column is blank.

https://github.com/cjimeneznow/parkingissuesimages/blob/master/img_3.7.3.png

a. In the debug output notice that we are also showing new output that looks like this:

17:41:44.637: TIME = 0:00:00.000 PATH = record/sys_portal_page/read CONTEXT = Enterprise CMDB Global RC = true RULE =

4. Open one of the records and notice that the employee values are not visible in the form. Let’s check out the debugging output to see if we can determine whether there are any issues that should be further reviewed.

a. Are there any excessively repetitive queries or security evaluation?

5. The added functionality that is leading to the behavior above may affect other functionality. Let’s use the record producer for the x_pir_employee_car table to see if we can observe the performance issue.

a. In the normal UI, impersonating go back to the /x_pir_employee_car_list.do list view if you are still in the form view. Click on the "new" ui action in the list view.

b. Notice any slowness. This may be related to the same issue as 4 above, but how?

c. If you switch to an admin user to you see the same slowness?


# Exercise #4 Upgrade Debugger
Imagine an upgrade just completed on the system and you have a Parking Spaces app. This app allows you to assign parking spaces to people, as long as they have the title, 'Manager'. After the upgrade, suddenly any title can have a parking space! Let’s debug.
1. In the **Normal UI**, navigate to **Parking Space > Parking Space Table**
2. Click **New** and choose an Employee. Notice you can Submit this record. Remember, only someone with a Manager title should be able to in this scenario.
3. We think something may have happened during the upgrade. Navigate to **System Diagnostics > Debug Upgrade**
4. Navigate back to **Parking Space Table** > **New**
5. Fill in the Employee, right click the top bar > **Save**
6. At the bottom of the page we have three new sections provided by the **Upgrade Debugger**
7. Expand all 3 sections and notice we have a **Business Rule, Confirm Employee Level** under **Customer Customized** and a **Script Include, CheckEmployeeLevel** under **ServiceNow Modified During Last Upgrade**. 
8. Click each of these records to open them in new tabs.
**NOTE:** At this point, a good idea would be to look at what changed to the Script Include and see if we were dependent on that change in the Business Rule, but since this is a mocked upgrade, the changes are not listed.
9. With the above note in mind, the change you *would have* seen is to **line 6**. **checkIfIsManager** changed to **checkIfManager**
10. Let's see if the Business Rule was calling the function by its old name. 
11. Update the Business Rule (you may have to click **here** at the top to edit the Business Rule) to call **checkIfManager**
12. Try to insert or update an existing record in **Parking Space Table** and notice you no longer can! (Unless they have the title **Manager**)