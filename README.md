To automate adding issues to a Beta GitHub Project using GitHub Actions, you need to create a Python script that interacts with the GitHub API, and then set up a GitHub Actions workflow to run this script when an issue is created.

### Step-by-Step Guide

#### 1. **Create the Python Script**

First, you need a Python script that uses the GitHub API to add issues to a Beta GitHub Project.

Save the following script as `add_issue_to_project.py`:

```python
import requests
import sys
import os

def add_issue_to_project(issue_number, project_id, column_id, token):
    # Get the issue details
    issue_url = f"https://api.github.com/repos/{os.getenv('GITHUB_REPOSITORY')}/issues/{issue_number}"
    headers = {
        "Authorization": f"token {token}",
        "Accept": "application/vnd.github.v3+json"
    }
    response = requests.get(issue_url, headers=headers)
    issue = response.json()
    
    # Add the issue to the project
    project_card_url = f"https://api.github.com/projects/columns/{column_id}/cards"
    card_data = {
        "content_id": issue["id"],
        "content_type": "Issue"
    }
    response = requests.post(project_card_url, headers=headers, json=card_data)
    response.raise_for_status()
    
    print(f"Issue {issue_number} added to project {project_id} in column {column_id}")

if __name__ == "__main__":
    issue_number = sys.argv[1]
    project_id = sys.argv[2]
    column_id = sys.argv[3]
    token = os.getenv('GH_TOKEN')
    
    add_issue_to_project(issue_number, project_id, column_id, token)
```

#### 2. **Store the Personal Access Token**

Store your GitHub Personal Access Token (PAT) as a secret in your repository:

1. Go to your repository on GitHub.
2. Navigate to `Settings` > `Secrets and variables` > `Actions` > `New repository secret`.
3. Create a new secret named `GH_TOKEN` and paste your PAT.

#### 3. **Create the GitHub Actions Workflow**

Create a workflow file in your repository at `.github/workflows/add-issue-to-project.yml`.

```yaml
name: Add Issue to Beta Project

on:
  issues:
    types: [opened]

jobs:
  add_issue_to_project:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.x'

      - name: Install Requests
        run: |
          python -m pip install --upgrade pip
          pip install requests

      - name: Run add_issue_to_project.py
        env:
          GH_TOKEN: ${{ secrets.GH_TOKEN }}
        run: |
          python add_issue_to_project.py ${{ github.event.issue.number }} <YOUR_PROJECT_ID> <YOUR_COLUMN_ID>
```

Replace `<YOUR_PROJECT_ID>` and `<YOUR_COLUMN_ID>` with the actual project and column IDs.

#### 4. **Retrieve Project and Column IDs**

You need to get the IDs of the Beta GitHub Project and the column where you want to add the issues.

##### a. Get Project ID

Use the GitHub API to list your projects and find the ID of the desired project.

```sh
curl -H "Authorization: token YOUR_PERSONAL_ACCESS_TOKEN" \
     -H "Accept: application/vnd.github.v3+json" \
     https://api.github.com/repos/your-org/your-repo/projects
```

##### b. Get Column ID

Use the GitHub API to list columns in the project.

```sh
curl -H "Authorization: token YOUR_PERSONAL_ACCESS_TOKEN" \
     -H "Accept: application/vnd.github.v3+json" \
     https://api.github.com/projects/YOUR_PROJECT_ID/columns
```

### Commit and Push

1. Save the Python script and the workflow file in your repository.
2. Commit and push the changes:

```sh
git add .github/workflows/add-issue-to-project.yml add_issue_to_project.py
git commit -m "Add workflow to add issues to Beta GitHub Project"
git push origin main
```

Now, when a new issue is created in your repository, the GitHub Actions workflow will run the Python script, which will add the issue to the specified Beta GitHub Project column.
