# airflow_installation_instructions repo
This repo contains a readme.md file explaining how to install Airflow locally using Docker. This is the fastest way to install Airflow on your computer. In this guide, I assume you already have Python installed on your computer. If you don't have Python installed, please install it from [python.org](https://www.python.org/)

# Steps to install Airflow locally, assuming you want to download version 2.3.4
1. Create a virtual environment → `python -m venv venv`

2. Make sure the python used is the one in the virtual env by running the command `which python`

3. Activate the virtual environment → `source venv/Scripts/activate` **(on Windows)** or `source venv/bin/activate` **(on Linux/Mac)**

4. **Package installations:**
    - `pip3 install apache-airflow==2.3.4`
    - Or also the following command ...
  ```
  pip3 install "apache-airflow[celery]==2.3.4" --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.3.4/constraints-3.7.txt"
  ```

5. Download **Docker** from this [link](https://www.docker.com/products/docker-desktop/)

6. Follow the steps in the official Airflow [documentation](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html) to install Airflow via Docker
    - Fetch docker-compose.yaml → `curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.4.0/docker-compose.yaml'`
    - Create a **.env** file in the **same directory** as **docker-compose.yaml** and add the following lines. You can also add any other environment variables that you need to use throughout your code (e.g., GOOGLE_APPLICATION_CREDENTIALS)
    - `AIRFLOW_IMAGE_NAME=apache/airflow:2.3.4`
    - `AIRFLOW_UID=50000`

7. Add a **Dockerfile** to your directory with the following lines. Write the file name like this without the double quotes “`Dockerfile`”, and VSCode will automatically detect that this is a Docker file)
  ```
  FROM apache/airflow:2.3.4
  # Install any dependencies that did not get installed with the official Airflow image
  # Make sure to pip freeze > requirements.txt first
  COPY requirements.txt .
  RUN pip3 install -r requirements.txt
  # You can create a folder in your local directory called py_scripts and put there any python modules that you want to import in your DAG
  # Make sure to include this line to add this folder to PATH in the local Airflow environment so that you can import your module right away with ease
  ENV PYTHONPATH="$PYTHONPATH:/opt/airflow/py_scripts"*/
  ```

8. If you want to send Emails through your DAG, add the following parameters under `_PIP_ADDITIONAL_REQUIREMENTS` in the **docker-compose.yaml** file
    - To generate the password, follow the steps explained in this [blog post](https://naiveskill.com/send-email-from-airflow/). Please create an environment variable in the .env file called GOOGLE_PASSWORD by simple typing `GOOGLEPASSWORD=fjkvbsjfls` and pass the variable to the docker-compose.yaml file as shown above
  ```
  # EMAIL CONFIG
  AIRFLOW__SMTP__SMTP_HOST: smtp.gmail.com
  AIRFLOW__SMTP__SMTP_PORT: 587
  AIRFLOW__SMTP__SMTP_USER: john.doe@gmail.com
  AIRFLOW__SMTP__SMTP_PASSWORD: ${GOOGLE_PASSWORD}
  AIRFLOW__SMTP__SMTP_MAIL_FROM: john.doe@gmail.com
  ```

9. Change `AIRFLOW__CORE__LOAD_EXAMPLES` to FALSE so that you **don't** load the default Airflow DAG examples and clutter your UI

10. Comment out this line in the docker-compose file `image: ${AIRFLOW_IMAGE_NAME:-apache/airflow:2.3.4}`
    - ... and uncomment this line `build: .`

11. Add more volumes (another name for folders) if you need to
  ```
  volumes:
    - ./dags:/opt/airflow/dags
    - ./logs:/opt/airflow/logs
    - ./plugins:/opt/airflow/plugins
    - ./py_scripts:/opt/airflow/py_scripts
  ```
  What you are doing here is mapping your local directories on the **left-hand side** to the Airflow directories on the **right-hand side**. `./dags`, `./logs`, and
  `./plugins` are the default volumes that get created when you run the docker-compose.yaml file
  - You can create a folder in your local directory called **py_scripts** and put there any python modules that you want to import in your DAG. If you do this, you need to add a another volume to the docker-compose.yaml file

12. Run `docker-compose up -d`
    - Make sure the Docker application is open and running in the background. You can always set it to open automatically as soon you log on to your computer so that         you don’t have to remember to open the application each time you run docker-compose
    
13. Run `docker ps` to check on the status of Airflow’s services. Everything should be healthy as shown below
![image](https://user-images.githubusercontent.com/98691360/192023711-09e5e2f4-3a2f-49b7-963c-cbab5604463b.png)

14. Now, go to your browser, type in **localhost:8080**. The username and password are both “`airflow`” without the double quotes. Congratulations, you are now running Airflow locally from your computer. It should look something like this  
![image](https://user-images.githubusercontent.com/98691360/192023824-8ae94ee0-7c1b-4a7d-917f-02b7c8ceb8a0.png)

# Manipulating the PYTHONPATH on Windows
To add a path to the PYTHONPATH on the Windows operating system, use the environment variables window from the start menu
![image](https://user-images.githubusercontent.com/98691360/192232307-bde284dc-c9fb-4087-871e-c505f363913d.png)

# Adding the GOOGLE_APPLICATION_CREDENTIALS to query data from BigQuery tables
By running this command, you will install the necessary libraries to query data from BigQuery tables. However, you will still need to configure the credentials so that
your requests don't get blocked
```
pip3 install "apache-airflow[celery]==2.3.4" --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.3.4/constraints-3.7.txt"
```
## Steps to set credentials:
1. Donwload [Google Cloud SDK](https://cloud.google.com/sdk/docs/install)

2. Open the Google Cloud SDK terminal after the installation finishes and type in the following commands one by one
    - ```gcloud auth application-default login --disable-quota-project```
    - ```gcloud config set project {insert_project_name_without_curly_braces}```
    - ```gcloud auth login --enable-gdrive-access --no-add-quota-project-to-adc```

3. Run docker-compose.yaml to activate the Airflow environment, go to **localhost:8080** after all services are spun up, click on "Admin --> Connections" to create
a connection and fill the fields as shown in the screenshot below
![image](https://user-images.githubusercontent.com/98691360/192107447-deab4d6b-277b-4e56-bac4-dc36cc0ae2cd.png)
    - The scopes that you need to enter in the **Scopes (comma separated)** field are shown below. Make sure to separate them by commas
        - https://www.googleapis.com/auth/drive
        - https://www.googleapis.com/auth/bigquery
        - https://www.googleapis.com/auth/cloud-platform

4. Go to the folder where your **application_default_credentials** are stored
    - On Windows, it's usually stored in this directory
        - ```C:\Users\%USER%\AppData\Roaming\gcloud\application_default_credentials.json``
    - On Mac/Linux, it's usually stored in this directory
        - ```$HOME/.config/gcloud/application_default_credentials.json```

5. Copy-paste the JSON file to one of the folders that get cloned to the Airflow environment. In this example, I use the folder **py_scripts**, which I created a 
volume for in **step #11** under the ```Steps to install Airflow locally``` section. You can also put in any of the other three volumes that are created by Airflow by default. In the end, it should look something like this

![image](https://user-images.githubusercontent.com/98691360/192233014-821f4c0f-968a-4b54-8391-a7420ee1d465.png)
    - **Important:** Remember to create a volume in the docker-compose.yaml that maps the folder where the JSON file is stored to the local environment in Airflow

6. At the beginning of the DAG script that is stored in the DAGs folder, add the following commands
```
import os
os.environ["GOOGLE_APPLICATION_CREDENTIALS"]="/opt/airflow/{volume_where_the_JSON_file_is_stored}/application_default_credentials.json"
```

7. Now, you should be able to query data in BigQuery through Airflow
