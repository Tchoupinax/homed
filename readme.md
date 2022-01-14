# homed

`homed` is a light-weight customizable portal primarily intended for the "self-hosted" crowd with built-in support for local authentication services (e.g. Authelia).  The user configures options and which links to display using a `yaml` file, and also has access to the HTML template (which uses [Jinja](https://jinja.palletsprojects.com/en/3.0.x/)), CSS, and JavaScript.  All resources are hosted locally.  Access to links is controlled via groups passed in by the authentication service using the `Remote-Groups` request header (which Authelia supports).

![homed](https://github.com/mwalters/homed/raw/main/screenshots/homed.png?raw=true "homed")

## Getting up and running
The easiest way to get up and running is with Docker.

Using `docker-compose`:

```
version: '2'
services:
  homed:
    image: mwalters/homed:latest
    container_name: homed
    ports:
      - 8020:5000
    volumes:
      - /local/directory:/config
    restart: unless-stopped
```

Using `docker-run`:

```
docker run -d \
  -p 8020:5000 \
  -v /local/directory:/config \
  --name homed \
  --restart=unless-stopped \
  mwalters/homed:latest
```

After initially running the container, you should be able to confirm it is running by visiting `http://your-host:8020`.  In your `/local/directory` you should now see an `app` folder.  Inside this folder is the code for `homed`.

* `yaml` configuration file is located in `app/homed.yaml` (see below for more information on configuration)
* Link images (e.g. icons for sites/services) can be added to `app/assets/logos`
* CSS is located in `app/assets/css/styles.css`
* HTML template being used is located in `app/templates/home.html`

## Configuration
Edit `app/homed.yaml`.  You should see this basic format:

```
name: homed
open_links_in_new_window: true
auth_ui_link: https://yourhost/authelia

sections:
  - Quick Links:
    name: "Quick Links"
    order: 1
    links:
      -
        name: Amazon
        link: https://smile.amazon.com/
        icon: /logos/amazon.svg
        authGroups:
          - users
  - Internal:
    name: "Internal"
    order: 2
    links:
      -
        name: "Dozzle"
        link: https://msw.io/dozzle/
        icon: /logos/dozzle.png
        authGroups:
          - admins
```

* `name` will be displayed in the title bar of the browser and at the top of the page
* `open_links_in_new_window` should be set to `true` or `false` depending on whether you want your links to open in a new browser window/tab or not
* `auth_ui_link` should be provided if you use an authentication provider such as Authelia.  This will be used to allow the user to log out or visit the Authelia page for whatever reason
* `sections` is a list of sections.  In the screenshot above, they are the containers that hold groups of links
    * `name` is the name for the section
    * `order` is the order the sections should be placed in on the screen (left to right, top to bottom)
    * `links` is a list of links to go in this section
        * `name` is the text to display in the link
        * `link` is the URL to send the user to when clicked
        * `icon` is the image to use for the link.  Some are provided in the image, but you can add your own by placing them in `app/assets/logos` and then referencing them here with `/logos/filename.ext`
        * `authGroups` is a list of groups passed in by your authentication service that you wish to have access to this link.  If the user does not belong to one of the groups specified, the link will not be displayed.  If no groups are provided here, then the links will be displayed to any/all users.

Add as many sections and links within those sections as you would like.

## Customization
As mentioned above, you have full access to the HTML (which supports [Jinja](https://jinja.palletsprojects.com/en/3.0.x/) templating), CSS, and JavaScript.  Feel free to edit it as you see fit, your changes should not be overwritten when the container is restarted.  If you remove a file that comes with the default installation, then it will be replaced when the container is restarted.

## Notes
* `homed` has no built in user system.  It is relying on your authentication provider to supply the expected headers: `Remote-User`, `Remote-Name`, `Remote-Email`, `Remote-Groups`.  These are provided by default with Authelia if you are using it with [SWAG](https://docs.linuxserver.io/general/swag).  If authentication information is not provided, then all links will be available to all users that can access the page.
