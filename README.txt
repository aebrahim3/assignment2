v1.18 - Authorization middleware
=================================
Although authorization sounds like authentication, they are both very different.
Authentication is about verifying that a user is who they claim to be.
This is often verifying their password matches what is in the database for that
username or email address. It may also include a fingerprint scan, a retina/iris
scan, a notification or text send to a phone or email adress or other methods of
Multi-factor authentication (MFA).

Authorization on the other hand is realizing that different users, depending on who
they are, should see different things. Administrators typically have full access
to everything the site, while users may only have the ability to edit their own
data, but only view everyone else's data. Guests may have even less permissions
and only have the ability to view the data and not edit anything. 

We will store the user's type in the session information so that we can check
it before we render the page. Using middleware, we'll ensure only admins can
view the admin page.

- Follow along instructions:

#Pick one user to promote to 'admin'
You can do this by going into Studio 3T and editing a user and setting their
user_type to "admin":

db.getCollection("users").updateOne({username: '<USERS_USERNAME>'}, {$set: {user_type: 'admin'}});

Note: replace <USERS_USERNAME> with the username of the user you want to promote to admin

- To Test:
Login using an admin user: http://localhost:3000/login
This page should be allowed: http://localhost:3000/admin
Delete your session cookie and refresh, you should be redirected to /login
Login using a normal user (non-admin): http://localhost:3000/login
This page should not be allowed: http://localhost:3000/admin

v1.17 - Authentication middleware
=================================
What is middleware and why should we use it?
Middleware are functions that we define that always get run when we get a request
for a page hit, but before we render the page. They are useful for parsing data
that we know that we might need (parsing the form fields in a POST is middleware
ex: app.use(express.urlencoded({extended: false})); ). We are also using middleware
to create and parse our cookies and handle our sessions for us. 
ex: app.use(session({...}));

Our site so far only has a few pages that require a login, but imagine a site
with dozens (or possibly even more) of pages where many of them require the user
to be logged in. We'd want a centralized single function to manage the login 
(to make the code more manageable and maintainable). Also, we'd want to make sure
we didn't overlook authentication when we create new pages - we wouldn't want to 
introduce a security risk, simply by forgetting to check for authentication.

By using middleware, we can say that all pages in a certain category (for example
everything under the /members folder must require a login). With a middleware 
authentication function, we can say any page hit in the the /members folder
must pass the authentication check before displaying the page.

- To Test:
Login using an existing user: http://localhost:3000/login
A valid password sends you here: http://localhost:3000/loggedin 
And this page should work too: http://localhost:3000/loggedin/info 
Delete the session cookie (from the client or server) and
both those sites shouldn't work.

v1.16 - Separating the Header and Footer
========================================
With templates we can also reduce redundancy. Notice how all of our pages
included the same code?  <html> <head> </head> <body> </body> </html>
The only thing that changes is all the code within the <body> tag.
So we can take everything before the body contents and move it to header.ejs
and everything after and move it to footer.ejs. After we've done this,
we can just include the header and footer in all our pages.
This makes it easier to update something in the header or footer 
for all the pages.

- To Test:
No functional changes have been made to the pages,
but we should probably make sure none of the pages are broken.
Test any page to make sure it works:
ex: Go to: http://localhost:3000/contact

v1.15 - Rendering Templates
===========================
In this commit we are adding an admin page to show all the users in the database.
We query the database and get all the users (we don't filter or sort them).
Then we render the admin page and pass the users array to the view.
In our admin.ejs we loop over the users array and include the user template.

- To Test:
Admin view of all users: http://localhost:3000/admin

- Notes:
There are a few problems with the admin page as it is now: (we'll fix these later)
1. It's not secure. Anyone can go to the admin page and see all the users.
2. Even if you were logged in, you should only be able to see the admin page if you are an admin.

v1.14 - Refactoring and more rendering
==============================
Let's replace all the res.send() calls with res.render.
This will mean in some cases, we'll need to pass parameters to the view,
for example the about page uses a query parameter to change the color
or the cat id to the cat page.

- To Test:
See that http://localhost:3000/about?color=red and 
http://localhost:3000/about?color=%239000C0 still work with the new view
(changing the color in the URL, changes the color of the text)

See that http://localhost:3000/contact and 
http://localhost:3000/contact?missing=1 still work with the new view
(should provide the email entry form and show "email is required" if the 
query parameter - missing=1 - is provided)
When you submit an email in the contact form, it should show your email here:
http://localhost:3000/submitEmail

The 404 page still works: http://localhost:3000/invalidpage

Check the cats:
http://localhost:3000/cat/1   => Fluffy
http://localhost:3000/cat/2   => Socks
http://localhost:3000/cat/3   => Invalid Cat id 3



- Suggestion:
Install the EJS language support Extension in VS Code 
for proper syntax highlighting.
Add this to your settings.json for shortcuts:
"emmet.includeLanguages": { "ejs": "html", }

v1.13 - EJS Template rendering
==============================
So far we've used res.send to construct and send html back to the client.
This really isn't scalable or manageable. It's also really bad practice.
You should separate your code from your HTML. Otherwise, you'll have a 
huge index.js file which continues to grow with every new page you create.

This is where we can separate our HTML pages into separate files, and do
some simple replacements for data and variables that might change those 
pages.

Enter EJS. EJS is a Simple templating language to help separate HTML from 
your code (while still making the HTML customizable and modular).

We put our HTML files in the views/ folder because that is the default
folder, but if you prefer a different folder, you can use this line to 
a different set of folders:
app.set('views', path.join(__dirname, '/yourViewDirectory'));

- Follow along instructions:
npm install ejs

- To Test:
open browser at: http://localhost:3000
You should see Hello World in large print (an <H1> tag)
 - This is no different from what it was before, but now the backend 
   is using EJS templates - much better!

v1.9 - Keeping secrets, secret with dotenv
==========================================
It's time we separate our passwords and secret keys from our code.
A password or secret_key should not be stored in a git repo.

We'll remove the passwords and secrets from the code and into a .env file.
Then we'll add a rule to our .gitignore file to exclude the .env file from entering our repo.

All secrets should be put in the .env file.
What is a secret?
 - Database Usernames and Passwords (as part of connection strings)
 - Encryption keys
 - API keys

- Follow along instructions:
npm install dotenv

Move secrets to the .env file
 *remove the enclosing quotes on string values
 *remove spaces before and after the equals symbol (=)
 *use uppercase for naming convention
 *no semicolon (;) at the end of a line
 *use can use a hash symbol (#) to make the rest of the line a comment
Ex:  DB_USERNAME=ElephantMan   #Comment: This user has read-only access to the DB
Ex2: DB_PASSWORD=A%r_bT90!KlBebv

- Notes:
When moving this code to production environments, you'll need to create
and populate the .env file on those systems (remember production will need
them, they aren't in the source code and otherwise your code will not run)

v1.8 - Sessions - With Encryption
===============================
Being able to read the session information on the server side is bad, 
because if someone gets access to the session database on mongodb, 
they can see session information. 

But fortunately, it's simple to encrypt it!!

- Follow along instructions:
Generate a new GUID for your mongodb_session_secret
(it should be different from your node_session_secret)
at: https://guidgenerator.com/
or: https://www.uuidgenerator.net/guid
or: https://www.guidgen.com/

- To Test:
Create a user http://localhost:3000/createUser
(remember to create some users - your server likely got rebooted and all users got deleted)

Attempt to log in http://localhost:3000/login
If you put in the same password it should go here: http://localhost:3000/loggedin

Open your mongo session database and you NOT be able to see the session information (usernames)
(the information IS still there, but now it's encrypted and impossible to read by hand 
and without the encryption key)

- Notes:
You should not store your mongodb username and password or your encryption secrets
in your source code and in your git repo.
 (again, we'll fix this later)

v1.7 - Sessions - No Encryption
===============================
In the previous "Login" version, we were supposed to go to /login,
enter our user and password and if our user was found _and_ the password was correct
it would let us go to /loggedin.
However, there was nothing preventing us from going to directly to /loggedin 
before logging in. Let's fix this with sessions.

Sessions store a cookie on the client side and a matching id on the server.
The server can store some data to confirm the user is properly logged in.
A valid session will only exist if you have logged in with the correct password (as an existing user)

I've chosen to store these sessions (the server side info) in a mongodb database,
but there are other ways to do this as well, MySQL, MS SQL, Redis, etc 
(See documentation for details: https://www.npmjs.com/package/express-session#compatible-session-stores)

I've set up a mongodb database for free on Atlas (https://www.mongodb.com/atlas/database)
Once setup, you can access your mongodb via the connection string Atlas provides you
ex: mongodb+srv://${mongodb_user}:${mongodb_password}@cluster0.ari8a.mongodb.net/${database_name}
(${mongodb_user} gets replaced with the user, ${mongodb_password} is replaced with the mongodb password
 and ${database_name} replaced with the database name)

- Follow along instructions:
npm install express-session
npm install connect-mongo
Generate your own GUID for your node_session_secret
at: https://guidgenerator.com/
or: https://www.uuidgenerator.net/guid
or: https://www.guidgen.com/

- To Test:
Create a user http://localhost:3000/createUser
(remember to create some users - your server likely got rebooted and all users got deleted)

Go directly to http://localhost:3000/loggedin, it should now say you aren't logged in
Attempt to log in http://localhost:3000/login
If you put in the same password it should go here: http://localhost:3000/loggedin
If not (wrong password or missing user) it goes back to login page: http://localhost:3000/login

Open your mongo session database and you can see your session with username stored in it (unencrypted)

- Notes:
Sessions are not encrypted on server side. This is BAD. :'(
You should not store your mongodb username and password in your source code and in your git repo.
 (again, we'll fix this later)

v1.6 - "Login" - using BCrypt to compare a user's password
==========================================================
If the passwords are no longer stored in plaintext, how do we tell if the 
password someone is entering when logging in is the same as when they created it?

BCrypt has a function to compare the current password with a previously 
hashed password - bcrypt.compareSync()

Let's use it see if we can "login" as user.

- To Test
Create a user http://localhost:3000/createUser
(remember to create some users - your server likely got rebooted and all users got deleted)
Attempt to log in http://localhost:3000/login
If you put in the same password it should go here: http://localhost:3000/loggedin
If not (wrong password or missing user) it goes back to login page: http://localhost:3000/login

- Notes:
We can access the logged in page (/loggedin) without actually going through the login 
process if we know the direct URL. :O This is BAD, and we'll fix it later

v1.5 - Hash passwords using BCrypt
==================================
Storing passwords in plaintext is BAD. 
VERY BAD.
Let's see how to use BCrypt to hash the passwords so no one can see them
(or steal them).

After hashing the passwords with BCrypt, they should look something like:
$2b$12$s2hrmuKearD75p4dkpTFBebvgGSKxeUOH51CwlnDHCmFcjjMIsuW.

- Follow along instructions:
npm install bcrypt

- To Test:
open browser at: http://localhost:3000/createUser
fill in a username and password, hit submit
You should be sent to /submitUser
And the user added. (Page shows a list of all users)
See the passwords are now hashed (and not plaintext)

- Notes: 
Even 2 people have the same password, their hashed passwords are different.
Try it out, create user1 with password "Password".
Create user2 with password "Password" and confirm their hashes are different.


v1.4 - Create a user using in-memory 'database'
===============================================
In this version we create an in-memory 'database'
where we store the users in an array, but it's only in the server's RAM
so if you restart node or reboot the computer, it gets erased (this is bad
but we'll fix this later).

/createUser  => form for entering username and password to create a new user

- To Test:
open browser at: http://localhost:3000/createUser
fill in a username and password, hit submit
You should be sent to /submitUser
And the user added. (Page shows a list of all users)
The list will grow for each time you post to add a new user

- Notes: 
If you reboot the server (restart nodemon) ALL USERS ARE DELETED! :O
Passwords stored in plaintext!! :O (this is VERY BAD!)

v1.3 - Form fields and POSTs
============================
How do we handle the text input when the user presses the submit button?

In this version we've created a page at /contact and it will
prompt for an email address. When you click on the button, it will
go to /submitEmail and acknowledge the email you entered.
If you didn't enter an email address, it will redirect you back
to /contact with a query parameter that will say the email address is required

- To test:
open browser at: http://localhost:3000/contact
try to hit the submit button (without an email address)
it should redirect to http://localhost:3000/contact?missing=1 
which will show an additional message about the email being required.
This time, put in an email address (ex: a@b.ca) and it will post you email to
http://localhost:3000/submitEmail and echo your email



v1.2 - Catch All and 404s
=========================
What happens if we go to a page that doesn't exist?

Previous versions will give an error message (http://localhost:3000/does_not_exist):
Cannot GET /does_not_exist

Let's fix this!

- To test:
open browser at: http://localhost:3000/does_not_exist
Should not say "Page not found - 404"
(also sends status 404)
Any route (page) not specified before the catch all will give a 404.

v1.1 - Simple Website playing with routes and URL and query params
==================================================================
I want to create a 2nd page at /about to show my name

- To test:
open browser at: http://localhost:3000/about
You should see Patrick Guichon in large print (an <H1> tag)

I want to have my name on the /about page change color
(using a URL parameter ex: /about?color=red OR /about?color=green OR /about?color=%239000C0)

- To test:
open browser at: http://localhost:3000/about?color=blue
You should see Patrick Guichon in blue.
Change the color URL parameter to change the color of the text

I want to display a few of my cats
/cat/1 and /cat/2
(using 'folders' as parameters)
pictures will be in a public folder. 
1 => fluffy.gif
2 => socks.gif

-To Test:
open browser at: http://localhost:3000/cat/1 
 see Cat 1 (fluffy.gif)
open browser at: http://localhost:3000/cat/2 
 see Cat 2 (socks.gif)
open browser at: http://localhost:3000/cat/3 
 see Error message that cat #3 doesn't exist

(You can also test by going to the images directly at http://localhost:3000/fluffy.gif and http://localhost:3000/socks.gif)

Image Credits: 
"fluffy" http://tenor.com/view/jinki948-cat-shocked-shocked-face-shock-gif-22585372
"socks" http://tenor.com/view/cat-shaking-leg-pukich-nasral-gif-18887227

v1.0 - Simple Website using Node.js
===================================

- Follow along instructions:
Steps:
1. npm init (all defaults are fine)
2. npm install
3. npm install express
4. create new file index.js


- Instructions to run:
npm install
nodemon index.js

- Git:
add a .gitignore to ignore all files in /node_modules

- To test:
open browser at: http://localhost:3000
You should see Hello World in large print (an <H1> tag)

- Notes:
You can go to the root, but no other sites
You will get an error for example if you go to /about or /login 
  (there's no code to handle these routes)