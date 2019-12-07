# Install and run anHALytics frontend

## Install required javascript libraries

Install first Bower, for instance on Ubuntu 

	> sudo npm install -g bower

Move to the anHALytics-frontend repository, for instance:

	> cd ~/anHALytics-frontend

Then, from there, run bower: 

	> bower install

Required javascript libraries will be installed. 

## Update the configuration

Open with your favorite editor the file ```js/resource/config.js``` and update the parameters, in particular the elasticsearch server host/port (exposing the different indexes built by the module anhalytics-core) and the (N)ERD server host/port for query disambiguation.

## Run the frontend

The frontend can then be run in any browser by exposing the project on a web server (for example : [http-server](https://www.npmjs.com/package/http-server)), or locally by opening the file ```index.html```.

To expose the GUI on a particular port using [http-server](https://www.npmjs.com/package/http-server), run from anHALytics-frontend root :

	> http-server -p 8061 -c-1
