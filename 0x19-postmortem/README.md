Postmortem Analysis
Following the release of ALX's System Engineering & DevOps project 0x19, an incident took place at approximately 06:00 West African Time (WAT) in Nigeria. The incident involved an outage on an isolated Ubuntu 14.04 container running an Apache web server. Instead of receiving the expected response of an HTML file defining a simple Holberton WordPress site, GET requests on the server resulted in 500 Internal Server Errors.

Debugging Process
Bug debugger Brennan (BDB... my actual initials, just made that up on the spot, pretty clever, huh?) encountered the issue at around 19:20 PST when opening the project and was tasked with addressing it. He promptly began the process of solving the problem.

To investigate, Brennan checked the running processes using the "ps aux" command. Two Apache processes, one under root and another under www-data, were found to be running correctly.

Next, he examined the sites-available folder in the /etc/apache2/ directory and determined that the web server was serving content located in /var/www/html/.

Brennan ran the "strace" command on the PID of the root Apache process in one terminal while using "curl" on the server in another terminal. Unfortunately, the "strace" output provided no useful information.

He repeated the previous step, but this time using the PID of the www-data process. With lowered expectations, Brennan was rewarded when the "strace" revealed an error (-1 ENOENT) indicating that the file "/var/www/html/wp-includes/class-wp-locale.phpp" did not exist.

Going through the files in the /var/www/html/ directory one by one, Brennan used Vim pattern matching to locate the erroneous ".phpp" file extension. He found it in the wp-settings.php file at line 137: "require_once( ABSPATH . WPINC . '/class-wp-locale.php' );".

Brennan removed the trailing "p" from the line and proceeded to test another "curl" request on the server, which returned a successful 200 response.

To automate the fix for this error, Brennan wrote a Puppet manifest named 0-strace_is_your_friend.pp.

Summary
In summary, the outage was caused by a simple typo. The WordPress application encountered a critical error in wp-settings.php when trying to load the file class-wp-locale.phpp. The correct file name, class-wp-locale.php, was located in the wp-content directory of the application folder. The patch involved a straightforward fix by removing the trailing "p" from the file name.

Prevention
This outage was not due to a web server error but rather an application error. To prevent similar outages in the future, please consider the following measures:

1. Testing: Perform thorough testing of the application before deployment. By conducting comprehensive tests, this error could have been identified and addressed earlier.

2. Status Monitoring: Implement a reliable uptime monitoring service like UptimeRobot to receive instant alerts in the event of a website outage.

It is worth noting that in response to this error, I wrote a Puppet manifest named 0-strace_is_your_friend.pp to automate the resolution of any identical errors that may occur in the future. The manifest replaces any "phpp" extensions in the file /var/www/html/wp-settings.php with "php."

However, rest assured that such errors will never occur again because we programmers never make mistakes! 😉                                                                   64,1          Bot
