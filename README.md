![animated demo of the client application](img/demo.gif)
<p align="center"><em>PackageMate: Self-hosted package tracking!</em></p>

**Note:** This is a fork of jat255's PackageMate with the primary difference being that it uses json files to store package info rather than a database.

This is a simple app using Json, Express.js, React, and Node to allow a user to create _package_ records and fetch their status from the carriers' APIs (or scraping, sometimes).

**Note:** This app was made as a full-stack app learning experience, so it certainly has some roughness around the edges, and I'm not responsible if it causes your packages to burst into flames... (additional note from crazybob1215: I'm adding additional jank to this by just changing the info storage methodology, so keep in mind that there are two layers of jank-ness going on)

It requires access to the carriers' API tools, which differs a bit for each carrier. The
following links have more information on how to create accounts and get credentials.
Some of the carriers have unstable (or unavailable) APIs, and so a web scraping approach
is used (currently for FedEx and Ontrac). This means these carriers take slightly longer to 
update, depending on how quickly the website responds, but they should eventually update.

## Tracking APIs:

USPS: 
  - https://www.usps.com/business/web-tools-apis/track-and-confirm-api_files/track-and-confirm-api.htm#_Toc41911503
  - Sign up here (free): https://www.usps.com/business/web-tools-apis/documentation-updates.htm
  - Once you get it, put your USPS "username" that they provide into `.env` in the proper place

UPS: 
 - https://www.ups.com/upsdeveloperkit/
 - Sign up here (free, but need a UPS account with payment method attached): https://www.ups.com/upsdeveloperkit/announcements
 - It took a while (about a day) when I signed up for my "account" to show in the list on the
   developer kit signup page, but once you sign up, place the UPS "Access key" into `.env`

Fedex: 
 - https://developer.fedex.com/api/en-us/home.html
 - Sign up here, create an "organization", then a new project (I named mine "PackageMate"),
   and then select the "Tracking API" and fill out the information they ask for. You will
   be given an "API Key", "Secret Key", and "Shipping Account" under the "TestKey" domain,
   but this Test key won't work for any real data. To access real tracking data, you need
   to move to "production", which requeires a Fedex Shipping account. Making an account
   (and the API access) is free, but does require a credit card. Follow the steps to open
   a "shipping account", and then link that account to your developer account. Once done,
   you'll get the real "API Key" and "Secret Key" in the production domain.
 - Put these values into the `.env` file in the proper place to enable the FedEx tracker.

OnTrac: 
 - Becuase of issues getting access to the API, OnTrac package status is obtained by 
   scraping the public website, so no API credentials are needed.

## Initial setup

The project is built using Node, and so can be run (with a few modifications) by using
the standard `npm start` approach (check the `package.json` files for more details about
the scripts that it uses).

Since there are a couple moving parts however, it is easier to get up and running using [Docker](www.docker.com) (specifically, [docker compose](https://docs.docker.com/compose/)). The `docker-compose.yml` file specifies two containers that will communicate with each other to run the app: a Node/Express server that does the API checks/scraping, and a React client application that you interact with in the browser.

### Clone or download the project using git:

```sh
$ git clone https://github.com/crazybob1215/PackageMate.git
$ cd PackageMate
```

Once this has downloaded, rename the `.env.example` file to `.env`, and replace the values 
indicated with ones that make sense for you (the two API credentials you obtained for
UPS and USPS).

### Build the Docker images and run the app

From the cloned folder, you should be able to run the app with one command:

```sh
$ docker-compose up --build -d
```

The first time this is run, it will take quite some time, since it will go through
and build the two docker images (and the server requires a number of dependencies
since `playwright` uses a headless version of Chromium for scraping). The `-d` flag
tells Docker to detach after bringing up all the containers. `--build` tells it
to build (or rebuild) the containers. Assuming all went well,
the app should be running in the background and should be accessible in your web
browser at http://localhost (or perhaps a different port if you changed it in
`docker-compose.yml`).

To view the logs of the application, check the browser console (for the client), or
run the following to see logs from each container in the app stack:

```sh
$ docker-compose logs --follow
```

To shut down the containers, run:

```sh
$ docker-compose down
```

The application data is stored in a [docker volume](https://docs.docker.com/storage/volumes/),
so to clear the app's data, you'll have to use docker tools. First find a list of the 
existing volumes using:

```sh
$ docker volume list
```

## Using the app

When you visit the app for the first time, there will be no packages in the system, so
the display will be empty:

![](img/new_install.png)

To add a package to the tracker, select the
correct carrier from the dropdown, paste the 
tracking number into the appropriate box, add a
description (if desired), and click the "Add 
Package" button (or press enter).

The package will show up in the table below, and
the app will immediately try to update the status
of the package. 

To update an individual package's status, click
the gray update button to the the right of the
table. Trackers using the scraping method (FedEx
and OnTrac, currently) will be significantly
slower than the direct API methods, unfortunately.
Be patient, and they should finish updating.

![app with some packages loaded in](img/active_packages.png)

To update all packages at once, click the blue
button above the table. The button will show
the progress of the operation to give an indication
of how many packages are left to track.

To stop tracking a package, click the red 
"archive" button for that package. The 
package will be moved to the "Archived" tab,
and will no longer be included in any updates.

## How to contribute?

Pull requests and feature additions are always welcome! In particular, 
PackageMate only supports four carriers, currently (the ones that I
receive packages through most often). If there's another carrier
that you would like to support, it should be added to the database
model in `models/package.js` (the `carrier` enum), the update function
in `routes/api.js`, and then in `3rd-party/trackers.js` and 
`3rd-party/parsers.js`.

### Testing trackers and parsers

Separate from the Docker and web application stack, there are some npm scripts
included to help debug/test the tracker and parser routines. A few examples
ar provided in the `package.json` such as the scripts `debug-fedex-tracker` and
`debug-fedex-parser`, which can be run from the command line with 
`$ npm run debug-fedex-tracker`. This helps accelerate the tracker development
process, as you don't have to wait for docker application to rebuild in order
to test the response (change the tracking number provided in the script to something
that makes sense to get it to work).
