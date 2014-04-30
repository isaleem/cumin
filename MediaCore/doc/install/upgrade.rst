.. _install_upgrade:

======================
Upgrading MediaDrop
======================

Upgrading MediaDrop is relatively straightforward:

1. Rename the old virtualenv directory (previously called `mediadrop_env`) and 
   create a new virtualenv.
2. Download and unpack the new version of MediaDrop (or fetch from github 
   and checkout a new tag in git if you used the git deployment).
3. If you previously kept your data folder and the config file
   (e.g. `deployment.ini`) inside the MediaDrop source code directory (as
   recommended in the docs before version 0.10) move them right next to the
   actual source code directory.
4. Update your configuration
5. Upgrade your database (completely automated)
6. Check your deployment scripts and post-upgrade cleanup

**NOTE 1:** `yourconf.ini` below should be replaced to refer to `development.ini`
or `deployment.ini`, depending on which one you have set up for your current
MediaDrop installation.

**NOTE 2:** `/path/to/venv` and `/path/to/mediadrop-old` below
should be replaced with the paths to your MediaDrop Virtual Environment
directory and Installation directory, respectively.

**NOTE 3:** `mediadrop-old` and `mediadrop-new` below should be replaced with
the correct versions for your installation. For example, with `MediaCore-0.9.4`
and `MediaCore-0.10.0`, respectively.

**NOTE 4:** If you're a developer type, and you're tracking the latest changes
using git, you'll want to run `paster setup-app development.ini` after fetching
any new changes. This will ensure that your custom appearance css and your
Players table are up to date.

Step 1: Re-Create your Virtual Environment
------------------------------------------

If you perform only a minor upgrade (e.g. MediaDrop 0.10.0 to 0.10.1) you 
can usually skip this step as we try not to change the database in a minor 
release.

.. sourcecode:: bash

   # Erase the old virtual environment:
   cd /path/to/venv
   cd ..
   mv venv venv.old

   # Create a new virtual environment:
   virtualenv --no-site-packages venv

   # Now, activate that virtual environment
   source venv/bin/activate

**NOTE 1:** Before MediaDrop 0.10 (aka MediaCore CE 0.10) the virtualenv folder was named 
'mediacore_env' in the documentation.

**NOTE 2:** If you kept your virtualenv inside the MediaDrop source code 
directory (as recommended in the docs before version 0.10) please create the 
new virtualenv next to (not inside) the source code folder (run ``cd ..`` 
before creating the new virtualenv).


Step 2: Install the new MediaDrop Files
------------------------------------------

`Download the latest official release of MediaDrop <http://mediadrop/download>`_ 
from our site, then unpack it beside your current MediaDrop installation.

.. sourcecode:: bash

   # Download the latest distribution beside the current installation
   cd /path/to/mediadrop-old/..
   wget http://static.mediadrop.net/releases/mediadrop-new.tar.gz

   # Unpack the downloaded distribution
   tar xzvf mediadrop-new.tar.gz

   # You should now have a mediadrop-old directory and mediadrop-new directory
   # inside your current directory. Check that this is the case:
   ls

   # Install MediaDrop and its dependencies into the new virtual environment
   cd mediadrop-new
   python setup.py develop


Step 3: Migrate your Data (Media, Thumbnails)
-----------------------------------------------

Depending on the version you are upgrading from, this step is a little different:

a. If you your data directory is not inside your MediaDrop source folder
   (recommended since MediaCore CE 0.10) you can skip this step.

b. If are upgrading from MediaCore **0.9.0 or newer**, you'll need to
   your data directory to a new place. It is recommended that you move the 
   directory and your config file (``yourconfig.ini``) right next (but not 
   inside) to your ``mediadrop-new`` directory.

   .. sourcecode:: bash

      # Navigate to the parent directory, where mediadrop-old and mediadrop-new
      # both reside.
      cd /path/to/mediadrop-old
      cd ..

      # Move over the old files (please note that there is no '/' after 'data')
      mv ./mediadrop-old/data .
      mv ./mediadrop-old/production.ini .

c. If you are upgrading from MediaCore CE **0.8.2 or older**, you'll need to
   move all of the relevant files in ``./data/`` and
   ``./mediacore/public/images/podcasts/`` and
   ``./mediacore/public/images/media/`` from your old installation directory.
   The new data folder should be in the same directory as (but not inside 
   of) ``mediacore-new``.

   .. sourcecode:: bash

      # Navigate to the parent directory, where mediacore-old and mediacore-new
      # both reside.
      cd /path/to/mediacore-old
      cd ..

      # Move over the old media files.
      mkdir ./data
      mv ./mediacore-old/data/deleted ./data/
      mv ./mediacore-old/data/media ./data/

      # Move over the old thumbnails.
      mv ./mediacore-old/mediacore/public/images/podcasts/[0-9]* ./data/images/podcasts/
      mv ./mediacore-old/mediacore/public/images/media/[0-9]* ./data/images/media/


Step 4: Update your configuration
---------------------------------

If you perform only a minor upgrade (e.g. MediaDrop 0.10.0 to 0.10.1) you 
can skip this step.

For major upgrades it is a good idea to create a new `deployment.ini` to check
for new configuration settings. If you upgrade from MediaCore CE 0.8.1 (or 
older) this is actually required.

.. sourcecode:: bash

    cp yourconf.ini yourconf-old.ini
    paster make-config MediaCore deployment.ini

Copying over any modifications you made to the old one. At the very least, 
this means you should be copying over the database configuration (a line that 
looks something like):

.. sourcecode:: ini

   sqlalchemy.url = mysql://mediadrop_user:mysecretpassword@localhost/mediadrop?charset=utf8&use_unicode=0


Step 5: Upgrade your database
-----------------------------

This step is slightly different depending on which version you are upgrading
from. See the individual commands below:

a.  If you're upgrading from **0.8.2, 0.9.0, or newer** (released after August 2010):

   .. sourcecode:: bash

      # Run the setup/upgrade script to upgrade your database.
      cd /path/to/mediadrop-new
      paster setup-app yourconf.ini

b.  Or, if you're upgrading from **0.8.0, 0.8.1** (released May 2010):

   .. sourcecode:: bash

      # Run the upgrade script to upgrade your database.
      cd /path/to/mediadrop-new
      python batch-scripts/upgrade/upgrade-from-v080.py yourconf.ini

c.  Or, if you're upgrading from **0.7.2** (released January 2010):

   .. sourcecode:: bash

      # Run the upgrade script to upgrade your database.
      cd /path/to/mediadrop-new
      python batch-scripts/upgrade/upgrade-from-v072.py yourconf.ini


Step 6: Check your deployment scripts and post-upgrade cleanup
--------------------------------------------------------------

If you already had MediaDrop deployed using mod_wsgi (:ref:`install_apache-wsgi`)
or mod_fastcgi (:ref:`install_apache-fastcgi`), you'll want to re-deploy using
the new installation. In particular, take note of changes to the deployment
configurations (e.g. Apache configuration), changed deployment scripts 
(e.g. `mediacore.wsgi`, `mediacore.fcgi`) and required file permissions.

If you didn't separate source code and data in your previous version of 
MediaDrop, you will need to adapt also the paths in your `mediacore.wsgi`
script (if you are using mod_wsgi).

If you recreated a new virtualenv in step 1 you also have to re-install any
plugins you have installed earlier.

When everything works fine your can also remove all the old directories 
`mediadrop-old` and `venv-old`).

post-upgrade cleanup for MediaDrop 0.10
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you're upgrading from MediaCore **0.9.x or smaller** and you use 
**Facebook comments** you have to run a special script to keep your existing
Facebook comments. Please read the release notes for more details.

.. sourcecode:: bash

      cd /path/to/mediadrop-new
      python batch-scripts/upgrade/upgrade_from_v09_preserve_facebook_xid_comments.py \
        --app-secret=<your-app-secret> yourconfig.ini


Done!
-----

Your migration to the latest MediaDrop is now complete.
