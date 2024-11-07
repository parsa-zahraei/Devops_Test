### Activity: Advanced GitHub Actions for a Flask Application

**Objective**: Build and enhance a simple Flask application, set up automated workflows with GitHub Actions for CI/CD, code quality checks, and optional deployment.

---

### Part 1: Set Up a Simple Flask Application

1. **Create a new folder** for the project and navigate into it.
2. **Set up a virtual environment** and install Flask:
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   pip install flask
   ```

3. **Create a `requirements.txt`** file to track dependencies:
   ```bash
   pip freeze > requirements.txt
   ```

4. **Add a `.gitignore` file** to exclude unnecessary files:
   - Create a `.gitignore` file with the following content:
     ```
     venv/
     __pycache__/
     *.pyc
     .DS_Store
     ```

5. **Create a basic Flask application** (`app.py`):
   ```python
   # app.py
   from flask import Flask, jsonify, request

   app = Flask(__name__)

   @app.route('/hello', methods=['GET'])
   def hello():
       return jsonify(message="Hello, World!")

   if __name__ == '__main__':
       app.run(debug=True)
   ```

6. **Write a test file** (`test_app.py`) for this endpoint:
   ```python
   # test_app.py
   import app

   def test_hello():
       client = app.app.test_client()
       response = client.get('/hello')
       assert response.status_code == 200
       assert response.get_json() == {"message": "Hello, World!"}
   ```

7. **Run the test** locally to confirm everything works:
   ```bash
   pytest test_app.py
   ```

---

### Part 2: Add the Project to GitHub and Set Up GitHub Actions

1. **Initialize a Git repository** and push it to GitHub:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
   git push -u origin main
   ```

2. **Create a GitHub Action for testing**:
   - In the repository, create a `.github/workflows/test.yml` file with the following content:
     ```yaml
     name: Python Application Test

     on: [push, pull_request]

     jobs:
       test:
         runs-on: ubuntu-latest

         steps:
           - name: Checkout code
             uses: actions/checkout@v4

           - name: Set up Python
             uses: actions/setup-python@v5
             with:
               python-version: '3.8'

           - name: Install dependencies
             run: |
               python -m pip install --upgrade pip
               pip install -r requirements.txt

           - name: Run tests
             run: pytest --cov=app test_app.py  # Run tests with coverage
     ```

3. **Commit and push** the workflow file. Confirm that the GitHub Actions workflow runs and passes in the Actions tab.

---

### Part 3: Add a Linter Workflow with Flake8

1. **Install Flake8** for code linting:
   ```bash
   pip install flake8
   ```

2. **Run Flake8** locally to check for linting issues:
   ```bash
   flake8 app.py test_app.py
   ```

3. **Create a separate GitHub Action for linting**:
   - Create a `.github/workflows/lint.yml` file with the following content:
     ```yaml
     name: Python Linting

     on: [push, pull_request]

     jobs:
       lint:
         runs-on: ubuntu-latest

         steps:
           - name: Checkout code
             uses: actions/checkout@v4

           - name: Set up Python
             uses: actions/setup-python@v5
             with:
               python-version: '3.8'

           - name: Install Flake8
             run: pip install flake8

           - name: Run Flake8
             run: flake8 app.py test_app.py
     ```

4. **Commit and push** the linter workflow file. Ensure the linter runs and passes on GitHub Actions.

---

### Part 4: Add Code Coverage Reporting

1. **Set up Codecov** to track test coverage.
   - Add `pytest-cov` to `requirements.txt`:
     ```bash
     pip install pytest-cov
     pip freeze > requirements.txt
     ```

2. **Modify the test workflow** to upload coverage results to Codecov (check the versions if not working (`v2`)):
   ```yaml
   - name: Upload coverage to Codecov
     uses: codecov/codecov-action@v2
     with:
       token: ${{ secrets.CODECOV_TOKEN }}  # Add Codecov token as a GitHub secret
   ```

3. **Commit and push**. Codecov will now track test coverage.

---

### Part 5: Add Dependency Management with Dependabot

1. **Enable Dependabot** to automatically create pull requests for dependency updates.
   - Create a `.github/dependabot.yml` file:
     ```yaml
     version: 2
     updates:
       - package-ecosystem: "pip"
         directory: "/"
         schedule:
           interval: "weekly"
     ```

2. **Commit and push**. Dependabot will check for outdated dependencies and open pull requests to update them.

---

### Part 6: Add Security Scanning with CodeQL

1. **Add a CodeQL analysis** workflow to check for security issues.
   - Create a `.github/workflows/codeql-analysis.yml` file (check the versions if not working (`v1`)):
     ```yaml
     name: "CodeQL"

     on:
       push:
         branches: [main]
       pull_request:
         branches: [main]

     jobs:
       analyze:
         name: Analyze
         runs-on: ubuntu-latest
         steps:
           - name: Checkout code
             uses: actions/checkout@v4
           - name: Initialize CodeQL
             uses: github/codeql-action/init@v1 
           - name: Perform CodeQL Analysis
             uses: github/codeql-action/analyze@v1
     ```

2. **Commit and push**. This workflow will now scan for security vulnerabilities in each pull request.

---

### Part 7: (optional) Set Up Notifications for Workflow Results

1. **Add Slack notifications** for workflow failures (optional).
   - In the test workflow file, add a Slack notification step:
     ```yaml
     - name: Slack Notification
       if: failure()
       uses: slackapi/slack-github-action@v1.16.0
       with:
         channel-id: "YOUR_CHANNEL_ID"
         slack-bot-token: ${{ secrets.SLACK_BOT_TOKEN }}
         text: "GitHub Action failed on ${{ github.repository }} at ${{ github.ref }}"
     ```

2. **Commit and push**. This will send Slack alerts on workflow failures.

---

### Part 8: Matrix Builds for Compatibility Testing

1. **Configure matrix builds** to test across multiple Python versions.
   - Modify the `jobs` section of the test workflow:
     ```yaml
     jobs:
       test:
         runs-on: ubuntu-latest
         strategy:
           matrix:
             python-version: [3.8, 3.9, 3.10]
         steps:
           # Rest of the steps...
     ```

2. **Commit and push**. The workflow will now test on multiple Python versions.

---

### Part 9: Add POST, PUT, and DELETE

Add other functionalities and see how the actions behave.

---

### Part 10: (optional) Deployment to AWS

1. **Set up AWS credentials** in GitHub secrets.
2. **Extend the GitHub Actions workflow** to deploy to AWS after successful tests.
