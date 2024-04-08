---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Getting Started

This section outlines the steps necessary to set up the Care Scribe feature within the Care application for both backend and frontend, as well as the integration of the Care Scribe plugin.

## System Requirements

To run the Care Scribe feature, the following system requirements must be met:

* Docker and Docker Compose
* Python 3.7 or higher
* PostgreSQL 10.0 or higher
* Redis
* Node.js 12.x or higher
* npm (compatible with the version of Node.js)

Ensure that your development environment meets these prerequisites before proceeding with the installation and setup.

## Installation Guide

### Backend - Using Docker Compose

It is highly recommended to use Docker Compose for setting up the Care application, which will consist of several Docker containers: `postgres`, `care`, `redis`, `celery`, and `localstack`.

#### Steps to run the development server using Docker:

1.  To start the development environment, execute:

    ```sh
    $ make up
    ```
2.  To load dummy data for testing, run:

    ```sh
    $ make load-dummy-data
    ```
3. Access the application at [http://localhost:9000](http://localhost:9000) in your browser.
4.  To stop the development environment, use:

    ```sh
    $ make down
    ```
5.  To run tests:

    ```sh
    $ make test
    ```

### Backend - Manual Setup

For manual setup, configure PostgreSQL and then follow the steps provided in the backend repository's documentation. This includes setting up the database, installing pre-commit hooks, and creating a superuser for the application.

#### Setting up PostgreSQL:

1. Initialize the database and set up the `postgres` user password.
2. Create a database named `care`.
3.  Run the following commands:

    ```sh
    export DATABASE_URL=postgres://postgres:<password>@127.0.0.1:5432/care
    python manage.py migrate
    python manage.py createsuperuser
    ```
4.  Use the command below to copy static files:

    ```sh
    python manage.py collectstatic
    ```
5.  To load dummy data for testing:

    ```sh
    python manage.py load_dummy_data
    ```

### Frontend Setup

To set up the frontend of the Care application, ensure that you have Node.js and npm installed, and then follow these steps:

1.  Install dependencies:

    ```sh
    npm install --legacy-peer-deps
    ```
2.  Run the app in development mode:

    ```sh
    npm run dev
    ```
3. Access the frontend at [localhost:4000](http://localhost:4000) in your browser.
4. Use staging API credentials provided in the documentation for authentication.

### Add Care Scribe Plugin to Care Backend

1. Clone the `care_scribe` repository into the same directory as the backend.
2.  Add it to the `INSTALLED_APPS` in the backend settings by including it in the `PLUGIN_APPS`:

    ```python
    PLUGIN_APPS = manager.get_apps()
    INSTALLED_APPS = DJANGO_APPS + THIRD_PARTY_APPS + LOCAL_APPS + PLUGIN_APPS
    ```
3.  Run the plugin installation script:

    ```sh
    python install_plugins.py
    ```

## Initial Setup

Once the installation is complete, configure the environment variables, database connections, and other necessary settings as outlined in the repository's documentation for both the backend and frontend parts of the application. After configuration, ensure all services are running correctly and the application is ready for use and further development.
