### Salesforce DX Hands-on Assignment 2

#### Develop from Existing Project with Salesforce DX

This assignment will teach you the scenario where you need to pick up where someone has left off. There is an ongoing project using Salesforce DX. As always, one of the developer in the team mistakenly left a bug in the project. Your job is to fix the bug and save the day.

#### Requirements
1. Clone from repository https://github.com/nutchanon-pho/hands-on-2.git
2. Create a new Git branch called `bug-fix-<your-name>`
3. Create a new Scratch Org and push the sources from this project to it
4. Try creating a new Account with the name `Beryl8` and see if the `Website` field is `www.beryl8.com`
5. You will see that it is not. Your job is to investigate and fix this bug.
6. To validate your result you can type `sfdx force:apex:test:run -l RunAllTestsInOrg -c -r human -w 1000` to run the test class.
7. Commit and push your bug fix to the branch `bug-fix-<your-name>`
Go to Github and send a pull request to the master branch
