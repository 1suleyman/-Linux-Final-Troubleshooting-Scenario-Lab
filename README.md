# 🐧 Linux Final Troubleshooting Scenario Lab

In this lab, I learned how to **transfer application files, extract and move directories, configure PostgreSQL access, update app settings, start a Python web app, and create a systemd service** for automatic startup and management.

---

## 📋 Lab Overview

**Goal:**

* Transfer application code from local laptop to dev servers
* Extract and place files in correct directories
* Configure PostgreSQL for remote access
* Fix database connection settings in the app
* Start the Python web application
* Create and enable a systemd service for the app

**Learning Outcomes:**

* Use `scp` to securely copy files between machines
* Use `tar` to extract archives
* Manage file ownership with `chown`
* Edit configuration files with `vi`
* Verify and manage PostgreSQL services
* Start a Django/Flask application
* Create and configure a custom `systemd` service

---

## 🛠 Step-by-Step Journey

### Step 1: Copy application archive to dev web server

```bash
scp -p /home/bob/calistone-code.tar.gz bob@devapp01:/home/bob
```

* Ensures the file is transferred to Bob’s home directory on `devapp01`.

---

### Step 2: Extract the archive on the dev server

```bash
ssh bob@devapp01
sudo mv /home/bob/calistone-code.tar /opt/
sudo tar -xf /opt/calistone-code.tar -C /opt/
ls /opt/calistone-code/mercury_project
```

* Confirms the project directory exists at `/opt/calistone-code/mercury_project`.

---

### Step 3: Verify PostgreSQL status on database server

```bash
ssh bob@devdb01
sudo systemctl status postgresql.service
```

* PostgreSQL initially inactive.

---

### Step 4: Configure PostgreSQL for remote access

* Add the following to `/etc/postgresql/.../pg_hba.conf`:

```text
host all all 0.0.0.0/0 md5
```

* Start the PostgreSQL service:

```bash
sudo systemctl start postgresql
sudo systemctl status postgresql
```

* Verify port 5432 is listening:

```bash
sudo netstat -natulp | grep postgres
```

---

### Step 5: Update application database settings

* Edit settings file (`settings.py`) to point to the correct DB host and port:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'HOST': 'devdb01',
        'PORT': '5432',
        ...
    }
}
```

---

### Step 6: Fix file ownership

```bash
sudo chown -R mercury: /opt/calistone-code
```

* Ensures all files and directories are owned by user `mercury`.

---

### Step 7: Start the application manually

```bash
cd /opt/calistone-code/mercury_project
source ../venv/bin/activate
python3 manage.py migrate
python3 manage.py runserver 0.0.0.0:8000
```

* Confirms the app is running and accessible on port 8000.

---

### Step 8: Create a systemd service for automatic management

**File:** `/etc/systemd/system/mercury.service`

```ini
[Unit]
Description=Project Mercury Web Application

[Service]
ExecStart=/usr/bin/python3 manage.py runserver 0.0.0.0:8000
WorkingDirectory=/opt/calistone-code/mercury_project
User=mercury
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

* Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now mercury.service
sudo systemctl status mercury.service
```

* Ensures the service starts automatically on boot and restarts on failure.

---

## ✅ Key Commands Summary

| Task                   | Command / Notes                                                 |
| ---------------------- | --------------------------------------------------------------- |
| Copy app archive       | `scp -p /home/bob/calistone-code.tar.gz bob@devapp01:/home/bob` |
| Extract archive        | `sudo tar -xf /opt/calistone-code.tar -C /opt/`                 |
| PostgreSQL status      | `sudo systemctl status postgresql.service`                      |
| Configure DB access    | Add `host all all 0.0.0.0/0 md5` to pg_hba.conf                 |
| Start DB               | `sudo systemctl start postgresql`                               |
| Update app settings    | `vi /opt/calistone-code/mercury_project/mercury/settings.py`    |
| Fix ownership          | `sudo chown -R mercury: /opt/calistone-code`                    |
| Start app manually     | `python3 manage.py runserver 0.0.0.0:8000`                      |
| Create systemd service | `/etc/systemd/system/mercury.service`                           |
| Enable/start service   | `sudo systemctl enable --now mercury.service`                   |

---

## 💡 Notes / Tips

* Always use `sudo` when moving files to `/opt` or editing system files.
* Verify project directories exist before running migration or starting the server.
* Use `netstat` to confirm PostgreSQL is listening on the correct port.
* Virtual environments must be activated before running Django commands.
* Systemd services simplify automatic startup and recovery after failure.

---

## 📌 Lab Summary

| Step                   | Status | Key Takeaways                     |
| ---------------------- | ------ | --------------------------------- |
| Transfer app archive   | ✅      | `scp` to dev server               |
| Extract archive        | ✅      | Moved to `/opt/calistone-code`    |
| Configure PostgreSQL   | ✅      | Remote access and port fixed      |
| Update app settings    | ✅      | Correct host/port                 |
| Fix ownership          | ✅      | All files owned by `mercury`      |
| Start app manually     | ✅      | Runs on 0.0.0.0:8000              |
| Create systemd service | ✅      | Auto-start and restart on failure |
