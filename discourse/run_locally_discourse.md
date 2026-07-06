Clone the Repository:
Download the Discourse source code and enter the directory.
`git clone https://github.com/discourse/discourse.git`
`cd discourse`

Initialize the Development Container:
Run the initialization script to set up the Docker image and create an admin user.
`d/boot_dev --init`
 - Enter your email and password when prompted.
 - Choose y to grant Admin privileges

Frontend start
 `d/dev`

backend start
`d/rails s` 

Visit http://localhost:4200.


Stop Discourse: `d/shutdown_dev`
Reset Database: `sudo rm -fr data`
Test Emails: Run `d/mailhog` in a third terminal.
Install Gems: Run `d/bundle install` if gems are missing.
