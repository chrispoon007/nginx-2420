# Setting Up a New Server on Arch Linux with Nginx

### Pre-Requisites:
- A server running Arch Linux (on DigitalOcean) with sudo access.

## Step 1: Installing Necessary Software

You will need to install Vim and Nginx on your Arch Linux server. You can do this using the following commands:

`sudo pacman -Syu vim nginx`

This command will update your package lists, upgrades your system, and installs Vim and Nginx.

## Step 2: Creating the Project Root Directory
Next, create a new directory that will act as your project root. This is where your website documents will be stored.

`sudo mkdir -p /web/html/nginx-2420`

## Step 3: Include `sites-enabled` in the `nginx.conf` File

Open the Nginx configuration file in Vim:

`sudo vim /etc/nginx/nginx.conf`

Add the following line to the end of the `http` block in the configuration file:

`include /etc/nginx/sites-enabled/*;`

Save and close the file with `:wq`.

## Step 4: Setting Up a Separate Server Block
You will need to set up a separate server block for your new server. This should not be part of the nginx.conf file. Instead, you should create a new configuration file under /etc/nginx/sites-available/ and link it to /etc/nginx/sites-enabled/.

First, create the following directories:

`sudo mkdir /etc/nginx/sites-available`
`sudo mkdir /etc/nginx/sites-enabled`

Next, create a new configuration file for your server block:

`sudo vim /etc/nginx/sites-available/nginx-2420`

In this file, add the following server block configuration:

```
server {
  listen 80;

  location / {
    root /web/html/nginx-2420;
    index index.html;
  }
}
```

Then, create a symlink to this file in the sites-enabled directory:

`sudo ln -s /etc/nginx/sites-available/nginx-2420 /etc/nginx/sites-enabled/`

## Step 4: Loading the Nginx Service and Enabling it on Boot

Start the Nginx service using the following command:
`sudo systemctl start nginx`

Ensure that Nginx is set to start on boot:
`sudo systemctl enable nginx`

You can check the status of the Nginx service using the following command:
`sudo systemctl status nginx`

If you need to reload the Nginx configuration, you can do so using the following command:
`sudo systemctl reload nginx`

## Step 5: Creating the Demo Webpage
Create a new HTML file in your project root directory:

`sudo vim /web/html/nginx-2420/index.html`

Paste the provided HTML code into this file, save, and close it.

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        * {
            color: #db4b4b;
            background: #16161e;
        }
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
            font-family: sans-serif;
        }
    </style>
</head>
<body>
    <h1>All your base are belong to us</h1>
</body>
</html>
```

Great Job! You should now be able to access your webpage by navigating to your server’s IP address (or localhost) in a web browser (e.g., http://137.184.89.16/).
