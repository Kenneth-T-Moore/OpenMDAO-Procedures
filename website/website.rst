
WebFaction
----------
	
The OpenMDAO website at ``openmdao.org`` is hosted by WebFaction.

.. note:: All URLs, usernames, and passwords for WebFaction and the other accounts listed below are available internally
          at ``havoc:/OpenMDAO/accounts_passwords.txt``.

	
Connection
==========
		
**SSH:**
		 
     `server:`  ``web39.webfaction.com``

     `username and password` are in a secure file that is currently on our internal server, Havoc.

**Control Panel:**  
			
     Control Panel login is at ``my.webfaction.com`` and uses the same login information as the SSH.

     `Domains:`  This is where we input ``openmdao.org`` and ``www.openmdao.org``.

     `Apps:` This is where all of the apps that run this website get routed to their part of the
     website and/or get their port numbers.

			
     .. image:: /images/website/apps.png

|    
 
  `Websites:` Makes sure that the site's apps have URLs that are directly reachable.  Here's the current
  mapping of site apps:

   .. image:: /images/website/urls.png
     
|   
 
     `Email addresses @openmdao.org:`  Under Emails, you can see how the ``@openmdao.org`` email addresses
     are set up.  Some are usernames, and others are aliases.  Currently all of these are forwarded to NASA
     addresses or gmail addresses as requested.  An email address can go to multiple people at once; for
     example, ``support@openmdao.org`` goes to both Keith and Justin.   


Organization/Filestructure
===========================

`Dists`    
~~~~~~~~

The ``dists`` directory is for holding local versions of packages that might need to be installed
(e.g., scipy) that can be gotten from our ``dists`` dir instead of having to go out to other
places to find them.  To add a new package, you must FTP the new file into the
``/home/openmdao/dists/`` directory at ``web39.webfaction.com``.   

Whenever a new package is added, the webpage at ``openmdao.org/dists`` needs to be updated to
reflect that addition. To do that, you must run Python 2.6 on the ``mkegglistindex.py`` file from
within the ``dists`` directory, like this:

::

  >> python2.6 /home/openmdao/dists/mkegglistindex.py

Once you have run the script, refresh the ``openmdao.org/dists`` webpage to make sure that the
update worked.  The newly added package should appear in the dists list.

`Downloads`
~~~~~~~~~~~  

The releases of OpenMDAO sit in this directory.  *Downloads* also has a script that creates the page
at ``openmdao.org/downloads``, and each version directory has its own page that needs to be
generated. From the instructions at ``/home/openmdao/downloads/README_TO_UPDATE``:

::

  TO UPDATE DOWNLOAD PAGES:

  First, cd into the most recent folder and then run "python2.6 dlversionindex.py".
  Then, cd back up to this level and run "python2.6 mkdownloadindex.py" in this directory.
  Finally, check openmdao.org/downloads/ to make sure that your efforts worked.

`Webapps`
~~~~~~~~~~

This section lists the webapps that ``openmdao.org`` uses. You will most likely need to refer to the information 
on ``custom_app`` and :ref:`OSQA`.

Custom_app
++++++++++++

This is a GitHub post-change hook. It's the trigger app that gets called automatically by GitHub whenever a change to
the ``dev``  branch of the repository is made.  This app parses out the XML from that commit and kicks off the
testing process.

When a pull request is approved on GitHub, it should trigger GitHub's ``post_receive`` hook.  The ``post_receive`` hook
should in turn contact the OpenMDAO testing application, which then turns on EC2 machines to test the most recent commit
against our test suite.  Sometimes, however, due to problems like the test server crashing, the ``post_receive`` fails to
start the testing.  In these cases, you'll need another event to trigger the ``post_receive`` hook, once the
underlying problem has been resolved (e.g., rebooting the test server.)  You don't want to have to do another commit to
trigger testing.  The event you need is the ``send_payload`` command, which works as follows:

**send_payload Command**

::

  -h, --help            
        show this help message and exit

  -c COMMIT_ID, --commit COMMIT_ID
        id of commit to test
	
  -r REPO, --repo REPO  
        repo url
  
  -b BRANCH, --branch BRANCH
        branch name
	
  -s SERVER, --server SERVER
        url:port of testapp server

The most common usage example of ``send_payload`` would look like this::

  send_payload -c [commit number] -s http://openmdao.org

If the ``send_payload`` usage is successful, an automated test will get kicked off and results will be posted to
http://openmdao.org/p_r.


**Updating and Restarting the Testserver**

The following procedure will properly update and restart the testserver:

1.  Connect to ``web39.webfaction.com`` using the openmdao account.

2.  Change directories into the ``custom_app`` repository with the command::

     cd webapps/custom_app/OpenMDAO-Framework


3.  Update the current repository by typing:: 

     git pull origin dev

4.  Remove the old ``devenv`` with the command::

     rm -rf devenv

5.  Build a new ``devenv`` with the command::

     python2.6 go-openmdao-dev.py

6.  Activate that new environment with the command::

    . /devenv/bin/activate


7.  Change directories into ``~/webapps/custom_app/openmdao_testapp`` directory. 

8.  Type::

     python2.6 setup.py develop

9.  Make sure that the previous testserver is no longer running. First, do a process listing using the command::

     ps -u openmdao
    
    Get the testserver's PID from that listing and then kill testserver by typing::
    
     kill -9 XXXX
    
    where XXXX is the PID.
    
10. To restart the test server, type::

     start_openmdao_testapp  

11. Exit web39


.. _`OSQA`:

OSQA
+++++

OSQA (Open Source Question & Answer) is an open source question-answer system written in Python with Django.

**Removing Spam Users**

A script has been written to remove spam users from the OSQA database. It is located in ``~/bin`` and can be run
from anywhere with the command::

  osqaDBclean.py  

+ *Arguments*

  :: 

    -h, --help 
          Show help message and exit 

    -v, --verbose 
          Enable verbose output 

    --nolog 
          Disable writing of log file 

    -u USERNAME, --username=USERNAME 
          The username to delete from the database 

    -f FILENAME, --file=FILENAME, --usernamefile=FILENAME 
          A file of usernames (separated by newlines) to delete 

    --sql 
          Make an .sql file of the database commands but do not execute 

    -a 
          Remove all suspended users from the database 

 
- *How to Use osqaDBclean.py*

 1. Create a backup of the database. Do this with the following command: 

    ::

      $ pg_dump -U database_name -f dump.sql 

   (The ``database_name`` is currently ``openmdao_osqa``.) 

 2. Run ``osqaDBclean.py`` with required arguments.


    .. Note:: You can run ``osqaDBclean.py`` with any of the options listed above, but you MUST specify either ``-f, -u,`` or
              ``-a``. You may use ``-f, -u,`` and ``-a`` together to specify multiple users to delete.


 3. Ensure the forums still work. If they do not, restore the database with the command:  

    ::

      $ psql -U database_name database_name < file  

 
- *How to Change the Database that osqaDBclean.py Connects to* 
 
  You must edit the script in order to change the database that it connects to. Find the following line (near the top of
  the file) and change the appropriate fields.  

  ::

    db = psycopg2.connect(host='127.0.0.1', 
    		database='openmdao_osqa', 
    		user='openmdao_osqa', 
    		password=?supersecretpassword',) 


  .. Note:: On WebFaction, ``database`` and ``user`` are ALWAYS the same. ``Password`` is not necessarily the same as
	    the ssh password. It is unique to the database and should not be changed without changing the password field
	    in the ``osqalocal_settings.py`` file.)


Procedures Doc
+++++++++++++++

The Procedure Doc is the document that you're reading now; it is kept on WebFaction under 
``/home/openmdao/docs/procedure_docs`` and points to the URL http://openmdao.org/procedures. That WebFaction folder is a
repository that watches ``git://github.com/OpenMDAO/OpenMDAO-Procedures.git``.  So when Procedures Doc repo is updated,  if
the changes are to be reflected in the online version, then you must go to this folder,  do a ``git pull`` to update the
repo, and then do ``make html`` to get the new doc built.

Stats
+++++++

This app populates a stats page up at ``openmdao.org/stats``.  It's a built-in WebFaction app, so you
can't do much other than install it and give it a URL. There's nothing to configure here, although
password-protecting this page could be useful.


WordPress
+++++++++

This app runs the bulk of the OpenMDAO website.  For details on WordPress, please see the following section.


WordPress 
----------

This tool is used to manage the information on the ``openmdao.org`` website. 

Content
=======

Most of the pages on the site are created as a `page` through the WordPress editor. The `front` page is a static HTML page.


**News** - The `News` page is a blog app plugin. Any `post` created in the WordPress editor shows up here. As the name implies, it should be used for news. 

**Downloads** - This is a family of pages. (`Downloads` leads to the downloads page that's generated by Justin's script.)

- **Recent Releases** and **Archives** pages are automatically generated. To add a release to the  downloads
  page, see the ``README_TO_UPDATE`` file in the ``downloads`` folder on the server.

- **Plugins** is simply a link to the GitHub repo.

- **Supported Operating Systems** is also automatically generated. This plugin (OpenMDAO Supported Systems
  Provider) grabs data from the Amazon EC2 machines to determine what OS, architecture, and Python version is
  being tested. To manually add a supported system, please see the ``README`` file in the plugin's directory.

**Support** - This is also a family of pages that take users to either documentation, screencasts, or to the OSQA app mentioned previously. 

- **Docs** and **Dev Docs** point to Sphinx documentation.

- **Forum** points to the OSQA forum. 

- **Screencasts** points to our YouTube page.

**Publications** - This is automatically generated from the ``publications`` folder on the server's home directory. Any file in that folder
will show up on the `Publications` page -- EXCEPT files that start with ``!``. File names must `not` contain spaces, and any underscores in the name will display as a space. See ``!README_TO_UPDATE.txt`` in the ``publications`` folder for more details. 

Changing the WordPress URL
=============================

1. Change the "app" path on ``my.webfaction.com``

 a) Go to ``my.webfaction.com`` and log in

 b) Navigate to ``Domains/Websites``

 c) Go to ``Websites``

 d) Click **edit** on the WordPress site

 e) Change the URL path of the ``wp_test`` app

2) In the ``functions.php`` of the current theme of the WordPress site (found in ``/wp-content/themes/'NAME-OF-THEME'/functions.php``), add two lines of code. 
   These should be the FIRST THING IN THE FILE, after ``<?php`` of course.

   ::

     define('WP_HOME','http://example.com');
     define('WP_SITEURL','http://example.com');

   If there is no ``functions.php`` file, create one with only those two lines. 

   Next, load the WordPress admin page until it works. 
   Log in and check to see that this is your site. 

   .. note:: Once your site is working, REMOVE THE LINES OF CODE FROM the ``function.php`` file.


3. Update the database (The image gallery will not work correctly until you do this.)

 a) Log in to the site's ``phpMyAdmin`` page, accessible from ``my.webfaction.com``. The password to the
    WordPress  database can be found under "Extra info" when clicking on the ``wp_test`` app from the
    **Applications** tab.

 b) Click on the WordPress database, and then click on the **SQL** tab on the top. Run the following code (replacing NEWURL with your new
    url, and OLDURL with your old url):

   ::

     update wp_posts set post_content = replace(post_content, "http://OLDURL.org", "http://NEWURL.org");
     update wp_options set option_value = replace(option_value, "http://OLDURL.org", "http://NEWURL.org");

   .. note:: Depending on your install, ``wp_posts`` and ``wp_options`` could have different prefixes. Adjust accordingly!


Updating the CSS or Header Art
================================

The website's CSS is defined by the current theme of the WordPress site. As of this writing, our theme is ``Yoko-OpenMDAO``
customization. Simply edit ``style.css`` as defined in the theme files to change our website's style. 

To change the header art, modify ``header.php`` in the current theme. The header art is loaded in the ``custom_banner`` div.


Amazon EC2
-----------

The Amazon Electronic Cloud Compute is where we host our machines that are involved in the automated online
testing.  The login info will be available in the password doc on Havoc. The process of setting up the machines is
discussed in a separate chapter of this document. Click `here <http://openmdao.org/procedures/amazon.html>`_ to 
view this information.


YouTube
-------

OpenMDAO has a YouTube account that is used for posting screencasts of installations and various things.  A
document on how to shoot a standard OpenMDAO screencast is HERE (link to the doc once it exists).  The email
address ``screencasts@openmdao.org`` is tied to this account and currently goes only to Keith.  We have a
`channel` at http://www.youtube.com/openmdao.  The username and password for this account will be in the
password document on Havoc.

Twitter
--------

OpenMDAO has a Twitter account that is used to announce new releases, new screencasts, or any other pertinent
news to our followers.  This is a simple one; simply use the login information to get into the account and
then post the pertinent information or reply to any direct mentions that may have happened.  Currently, the
Twitter account is tied to the ``support@openmdao.org`` email address, so if you want to be copied on Twitter
notifications, add yourself to that email address (see above section on email aliases). Our feed is available
at: http://twitter.com/#!/openmdao.  The username and password for this account will be in the
password document on Havoc.


Launchpad
----------

``launchpad.net/openmdao`` is no longer used but has a re-direct to the current project site and to
GitHub.  The only way to control this stuff is through Keith's account.


GitHub
-------

**Service Hooks:** GitHub is great for keeping code repositories, housing issues (formerly known as
tickets in our Trac world), and hosting wiki pages.  But for the Framework repository, we also have
a post-commit hook set.  Whenever a commit occurs on the dev branch, a blast of XML is sent to the
``custom_app`` we have running on WebFaction.  That app in turn kicks off the build and uses the XML
to log info on the commit that triggered the build.  

This process is wired together on GitHub at: https://github.com/OpenMDAO/OpenMDAO-Framework/admin.
(This link works only if you have admin privileges.)

Click **Service Hooks** in the left-hand menu.

Then click **Post-Receive URLs.** 

At this point, you'll be able to edit the URL or turn off the service completely.

.. note:: The **Twitter** service hook is currently turned off because commit chatter is too high. Despite
	  being off, the hook is wired to work with just a simple activation of an "active"  check box.

GoDaddy.com
------------

``GoDaddy.com`` handles our domain names and forwards them to WebFaction.

**Names:** ``openmdao.org``  (``openmdao.net, openmdao.com,`` and ``openmdao.info`` are set up to redirect to ``www.openmdao.org``) 

**Renewal:** Domain names are held until 10/24/2018.

**Tying to WebFaction:** In the GoDaddy account, the nameservers ``NS1.WEBFACTION.COM`` (NS1 through NS4) are
used.
