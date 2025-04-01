# Supabase Inactive Fix

![GitHub stars](https://img.shields.io/github/stars/travisvn/supabase-inactive-fix?style=social)
![GitHub forks](https://img.shields.io/github/forks/travisvn/supabase-inactive-fix?style=social)
![GitHub repo size](https://img.shields.io/github/repo-size/travisvn/supabase-inactive-fix)
![GitHub language count](https://img.shields.io/github/languages/count/travisvn/supabase-inactive-fix)
![GitHub top language](https://img.shields.io/github/languages/top/travisvn/supabase-inactive-fix)
![GitHub last commit](https://img.shields.io/github/last-commit/travisvn/supabase-inactive-fix?color=red)
![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Ftravisvn%2Fsupabase-inactive-fix&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)
[![](https://img.shields.io/static/v1?label=Sponsor&message=%E2%9D%A4&logo=GitHub&color=%23fe8e86)](https://img.shields.io/github/sponsors/travisvn)

This project helps prevent Supabase projects from pausing due to inactivity by periodically inserting, monitoring, and deleting entries in the specified tables of multiple Supabase databases. The project uses a configuration file (`config.json`) to define multiple databases and automate the keep-alive actions.

## Features ⭐️

- Insert a random string into a specified table for each Supabase database.
- Monitor the number of entries in the table.
- Automatically delete entries if the table contains more than a specified number of records.
- Log successes and failures, and generate a detailed status report.

## Setup 🚀

1. Clone the repository:
    
    ```bash
    git clone https://github.com/travisvn/supabase-inactive-fix.git
    cd supabase-inactive-fix
    ```
    
2. Install the required dependencies:
    
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows use `venv\Scripts\activate`
    pip install -r requirements.txt
    ```
    
3. Create a `config.json` file in the project root. This file defines your Supabase databases. 

   
    Example configuration:
    
    ```js
    [
      // Use both environment variable for url and key
      {
        "name": "Database1",
        "supabase_url_env": "SUPABASE_URL_1",
        "supabase_key_env": "SUPABASE_KEY_1",
        "table_name": "KeepAlive"
      },
      // Use both plain text variable for url and key
      {
        "name": "Database2",
        "supabase_url": "https://your-supabase-url-2.supabase.co",
        "supabase_key": "your-direct-supabase-key",
        "table_name": "keep-alive"
      },
      // Mix use of env and plain text is also allowed
    ]
    ```

    [See the section below for how to easily configure your database](#supabase-database-setup)
    
    ### Environment Variables Explained
    
    In the `config.json` file, you can define either:
    
    - **Direct URL and API Key**: Use the `"supabase_url"` field to directly specify your Supabase API Url. Use the `"supabase_key"` field to directly specify your Supabase API key.
    - **Environment Variable**: Use the `"supabase_key_env"` field to reference an environment variable where the key is stored. Use the `"supabase_url_env"` field to reference an environment variable where the url is stored. This is more secure, especially when running the script in different environments.
    
    #### Example:
    
    - `"supabase_key_env": "SUPABASE_KEY_1"`: This tells the script to look for an environment variable called `SUPABASE_KEY_1` that contains the actual API key.
    - `"supabase_key": "your-direct-supabase-key"`: This directly provides the API key within the `config.json` file, which is less secure but simpler for local setups.

4. Set up your environment variables _if you're using them_:
    
    Create a `.env` file and store variables there
    
    ```
    SUPABASE_URL_1="your-supabase-url-1"
    SUPABASE_URL_2="your-supabase-url-2"

    SUPABASE_KEY_1="your-supabase-key-1"
    SUPABASE_KEY_2="your-supabase-key-2"
    ```
    
5. Run the script:
    
    ```bash
    python main.py
    ```

## Supabase Database Setup 🔧

This project is predicated on accessing a `keep-alive` table in your Postgres database on Supabase. 

### Sample SQL 

Here's a SQL query for a `keep-alive` table 

```sql
CREATE TABLE "keep-alive" (
  id BIGINT generated BY DEFAULT AS IDENTITY,
  name text NULL DEFAULT '':: text,
  random uuid NULL DEFAULT gen_random_uuid (),
  CONSTRAINT "keep-alive_pkey" PRIMARY key (id)
);

INSERT INTO
  "keep-alive"(name)
VALUES
  ('placeholder'),
  ('example');
```
    

## Cron Job Setup ⏱️

To automate this script, you can create a cron job that runs the script periodically. Below are instructions for setting this up on macOS, Linux, and Windows.

### macOS/Linux

1. Open your crontab file for editing:
    
    ```bash
    crontab -e
    ```
    
2. Add a new cron job to run the script every Monday and Thursday at midnight:
    
    ```bash
    0 0 * * 1,4 cd /path/to/your/project && /path/to/your/project/venv/bin/python main.py >> /path/to/your/project/logfile.log 2>&1
    ```
    

This example cron job will:

- Navigate to the project directory.
- Run the Python script using the virtual environment.
- Append the output to a logfile.

For reference, here’s an example used in development:

```bash
0 0 * * 1,4 cd /Users/travis/Workspace/supabase-inactive-fix && /Users/travis/Workspace/supabase-inactive-fix/venv/bin/python main.py >> /Users/travis/Workspace/supabase-inactive-fix/logfile.log 2>&1
```

### Windows (Task Scheduler)

Windows does not have cron jobs, but you can achieve similar functionality using Task Scheduler.

1. Open **Task Scheduler** and select **Create Basic Task**.
    
2. Name the task and set the trigger to run weekly.
    
3. Set the days (e.g., Monday and Thursday) and time (e.g., midnight) when the script should run.
    
4. In the **Action** step, select **Start a Program**, and point it to your Python executable within your virtual environment. For example:
    
    ```vbnet
    C:\path\to\your\project\venv\Scripts\python.exe
    ```
    
5. In the **Arguments** field, specify the path to the script:
    
    ```vbnet
    C:\path\to\your\project\main.py
    ```
    
6. Save the task. The script will now run automatically according to the schedule you specified.
    
### GitHub Actions Cron Setup

A simple cron job is already configured in `.github/workflows/cron.yaml` and will:
- Set up Python environment
- Install dependencies
- Create config and .env files
- Run the keep-alive script
- Upload logs if the job fails

**Usage:**

1. Go to your repository's **Settings** > **Secrets and variables** > **Actions**
2. Add the following repository secrets:
   - `SUPABASE_URL_1` with your Supabase URL
   - `SUPABASE_KEY_1` with your Supabase key
   - _Feel free to add more based on your config, and you will need to adjust `cron.yaml` config generating section._
3. Add a repository variable:
   - `ENABLE_CRON` set to `true`
4. The workflow is scheduled to execute automatically every Monday and Thursday at 00:00 UTC. Additionally, you can manually trigger the workflow by navigating to the Actions tab and selecting "Run workflow". Modify the schedule as needed to suit your requirements.

## Contribution

Feel free to open an issue or submit a pull request if you'd like to contribute to this project.

## License

This project is licensed under the MIT License. See the `LICENSE` file for more details.
