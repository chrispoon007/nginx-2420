# Tutorial: Adding Firewall, Backend, and Reverse Proxy to Nginx Server on Arch Linux

This tutorial will walk you through how to secure your server with a firewall and introduce a backend service with Nginx that acts as a reverse proxy.

*Note: This tutorial assumes you have already set up Nginx (including the Nginx config file) on your Arch Linux server.*

### Prerequisites

* Functional Nginx Server with the basic Nginx configuration set up
    - Update/Install nginx with `sudo pacman -S nginx`
* Access to your server via SSH - `ssh username@serverIP`
* `hello-server` binary downloaded to your local machine

### Installing and Setting Up the UFW Firewall

1. **Install UFW:**

    `sudo pacman -S ufw`

2. **Allow SSH (Otherwise you lose SSH connection), HTTP, and HTTPS ports through the firewall:**

    ```
    sudo ufw allow 22
    sudo ufw allow 80
    sudo ufw allow 443
    ```

3. **Enable UFW:**
   `sudo ufw enable`

4. **You can check the status of UFW and see the added rules with this command:**
   `sudo ufw status`  

### Setting Up the Backend Service

1. **Download the binary and move it to your server:**

   Transfer the `hello-server` binary to your `/usr/local/bin` on your server. This is a common location for system binaries. 

   You can use the `sftp` command to transfer the file from your local machine to your linux server:
   
   `sftp chris@137.184.89.16` # Replace with your username and server IP
   
   then write `put [path to your directory]/hello-server` to transfer the file to your linux server home directory.
   
   Then use `sudo mv` to move the file to `/usr/local/bin`.

2. **Create a Systemd Service File:**

   Create a new file named `hello-server.service` in the systemd service unit directory:
   
   `sudo vim /etc/systemd/system/hello-server.service`

   Paste the following content into the file:

   ```
   [Unit]
   Description=Hello Server Service

   [Service]
   ExecStart=/usr/local/bin/hello-server 
   Restart=always       # This will make it so that the service will restart in case it crashes or stops

   [Install]
   WantedBy=multi-user.target
   ```

   Save and close the file (:wq)

3. **Start and Enable the Service at Start:**

   First, we must reload systemd config with daemon-reload:
   
   `sudo systemctl daemon-reload`

   Now, start and enable the service:
   
   `sudo systemctl start hello-server.service`
   
   `sudo systemctl enable hello-server.service`

4. **Verify Service Status:**

   `sudo systemctl status hello-server.service`

   This should show the service as running (active).


### Configuring the Nginx Reverse Proxy

1. **Edit the Server Block:**

   Open the Nginx server block configuration file (that was created in the previous Nginx setup):
   
   `sudo vim /etc/nginx/sites-available/nginx-2420`

2. **Add Reverse Proxy Configuration:**

   ```
   server {
       listen 80;

       location / {
           root /web/html/nginx-2420;
           index index.html;
       }

       location /hey {
           # Reverse Proxy for /hey
           proxy_pass http://127.0.0.1:8080/hey;
       }

       location /echo {
           # Reverse Proxy for /echo
           proxy_pass http://127.0.0.1:8080/echo;
       }
   }
   ```

    Save and close the file (:wq)

3. **Start and Enable Nginx:**

    `sudo systemctl start nginx`
    
    `sudo systemctl enable nginx`

4. **Test and Reload Nginx Configuration:**

    `sudo nginx -t`

If you make any changes to the Nginx configuration, you can reload the configuration with:

    `sudo systemctl reload nginx`

5.  **Test the Backend**

You can test the backend connection using curl. From your host machine (not the server machine), run:

`curl http://137.184.89.16/hey`
and
`curl -X POST -H "Content-Type: application/json" -d '{"message": "Hello from your server"}' http://137.184.89.16/echo`

Replace the IP address with the IP address of your server machine.

If everything is working correctly, you should see the response from the backend server (`Hey there & {"message": "Hello from your server"}`
or similar, respectively).

That’s it! You have now set up a firewall using UFW and a reverse proxy server to the backend on your Arch Linux droplet.
   