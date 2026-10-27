# Flask Web App with Nginx and systemd

## Goal
Run a small Python web app that reads from the MariaDB `students` table, keep it running as a service, and serve it through Nginx.

## Concept
- **Flask** is a Python framework for building web apps.
- **systemd** manages background services, so the app starts on boot and restarts if it crashes.
- A **reverse proxy** (Nginx) receives web requests and passes them to the app. This is the standard way to deploy web apps.

## Steps
1. Set up Python and installed Flask plus a MariaDB connector (TODO: which package):
   ```bash
   sudo apt install python3-venv
   python3 -m venv venv
   source venv/bin/activate
   pip install flask   # TODO: add the database package you used
   ```
2. Wrote the app, which queries the `students` table and shows the results at `/students`.
   ```python
   # TODO: paste your app.py here (remove any real passwords)
   ```
3. Created a systemd service named `webapp` so the app runs in the background:
   ```ini
   # /etc/systemd/system/webapp.service
   # TODO: paste your actual file. A typical one looks like:
   [Unit]
   Description=Flask web app
   After=network.target mariadb.service

   [Service]
   User=<username>
   WorkingDirectory=<path to app>
   ExecStart=<path to venv>/bin/python app.py
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable --now webapp
   systemctl status webapp
   ```
4. Added a reverse proxy in the Nginx config so `/students` goes to the app:
   ```nginx
   # TODO: paste your actual location block. Example:
   location /students {
       proxy_pass http://127.0.0.1:<app port>;
   }
   ```
   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```
5. Opened `http://localhost:8080/students` from Windows and saw the table data.

![Students page in browser](../screenshots/06-students-page.png)

## Problems and fixes
- TODO: describe any problems (for example, a service failing to start and how `journalctl -u webapp` helped). If none, write "None encountered."

## What I learned
- TODO: one or two sentences.
