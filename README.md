# Vocabulary Bank Update Automation

## Project Summary
This project aims to automate the process of updating a vocabulary bank (`eng_vocab_bank.json`) by using GitHub Actions. Whenever new vocabulary entries are added to a temporary file (`temp_note.json`), the workflow automatically appends these entries to the main vocabulary bank. This ensures that the vocabulary bank is always up-to-date without requiring manual intervention.

## Overview
This document outlines the design and steps for automating the update of `eng_vocab_bank.json` using GitHub Actions. When a new vocabulary is added to `temp_note.json`, the automation process will ensure that it is appended to the vocabulary bank file `eng_vocab_bank.json`. The update process will be triggered whenever `temp_note.json` is updated.

## Design
1. **Trigger**: The workflow is triggered by a `push` event whenever `temp_note.json` is updated in the repository.
2. **Steps in the Workflow**:
   - **Checkout the Repository**: Use `actions/checkout` to get the repository files into the workflow environment.
   - **Set Up Node.js**: Install Node.js to enable the processing of JSON files.
   - **Update Vocabulary Bank**: Read `temp_note.json` and append its contents to `eng_vocab_bank.json`.
   - **Commit and Push Changes**: Commit the updated `eng_vocab_bank.json` and push it to the repository.

## GitHub Actions Workflow
Below is the YAML configuration used to set up the GitHub Actions workflow:

```yaml
yaml
name: Update Vocabulary Bank

on:
  push:
    paths:
      - 'temp_note.json'

jobs:
  update-vocabulary:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14'

    - name: Install dependencies
      run: npm install

    - name: Add temp_note.json data to eng_vocab_bank.json
      run: |
        node <<EOF
        const fs = require('fs');
        const tempNote = JSON.parse(fs.readFileSync('temp_note.json'));
        const vocabBankPath = 'eng_vocab_bank.json';

        let vocabBank = [];
        if (fs.existsSync(vocabBankPath)) {
          vocabBank = JSON.parse(fs.readFileSync(vocabBankPath));
        }

        vocabBank = vocabBank.concat(tempNote.vocabulary);
        fs.writeFileSync(vocabBankPath, JSON.stringify(vocabBank, null, 2));
        EOF

    - name: Commit and push changes
      run: |
        git config --local user.name "github-actions"
        git config --local user.email "github-actions@github.com"
        git add eng_vocab_bank.json
        git commit -m "Add vocabulary from temp_note.json"
        git push origin HEAD
```

## Steps to Implement
1. **Create Repository and Branch**: Create a new repository and make a test branch where you want to automate the updates.
2. **Add Files**: Add `temp_note.json` and `eng_vocab_bank.json` to the repository.
3. **Create GitHub Actions Workflow**: Create a `.github/workflows` directory in your repository, and add a YAML file with the GitHub Actions workflow provided above.
4. **Test the Workflow**: Commit changes to `temp_note.json` to trigger the workflow and verify that `eng_vocab_bank.json` is updated automatically.

## Notes
- Ensure that the repository permissions allow GitHub Actions to push changes back to the repository.
- You can modify the commit message or user details (`user.name` and `user.email`) as needed.
- The workflow uses Node.js to manipulate JSON data, so Node.js must be set up properly during the workflow execution.

## Future Improvements
- **Error Handling**: Add error handling in the Node.js script to manage cases where the JSON is improperly formatted.
- **Notification**: Implement a notification mechanism (e.g., Slack, email) to notify when the vocabulary bank is successfully updated.

