---
name: check-leaves-balances
description: check-leaves-balances skill to check available leaves across all leave types
---



# Pre-Requisites
- Always work in current project root directory
- DevOn's Greythr Portal credentials must be set as environment variables `GREYTHR_EMPLOYEE_ID` and `GREYTHR_PASSWORD`. Never ask the user for credentials directly.

```bash
pwd && if [[ -n "$GREYTHR_EMPLOYEE_ID" && -n "$GREYTHR_PASSWORD" ]]; then echo "CREDENTIALS_SET"; else echo "CREDENTIALS_MISSING"; fi
```
- If `CREDENTIALS_MISSING` is printed, exit the workflow and ask the user to run `export GREYTHR_EMPLOYEE_ID=<your_employee_id>` and `export GREYTHR_PASSWORD=<your_password>`, then restart the coding agent session/terminal.

- Install `browser-act` skill using the command 
```bash
npx skills add https://github.com/jayoswal-devon/devon-greythr-skills --skill browser-act
```
- After installing read the `browser-act` skill before proceeding further.

**This is a strict requirement and cannot be avoided, even if they seem insignificant, they are crucial for the proper functioning of the skill.**

**Do NOT skip this step regardless of how simple the workflow seems.**

# Workflow
1. check browser is available using `browser-act browser list`
  1.a If not available then create a new browser
  1.b If available then use the existing browser
2. Session management, always create new session for each workflow run and close the session after the workflow is completed.
3. Goto url `https://devon.greythr.com`
4. Enter employee_id and password from environment variables `GREYTHR_EMPLOYEE_ID` and `GREYTHR_PASSWORD` respectively and click on login button.
5. After successful login, on the left side panel click on the `Leave` dropdown
6. Click on `Leave Balances` option from the dropdown
7. Take a screenshot and try to extract the available leaves balances across all leave types and present it to the user in a tabular format.
  7.a if unable to extract information by image, then understand from the browser state and extract the information from the page and present it to the user in a tabular format.
8. Click on logout button (present at top right corner) to logout from the portal and close the browser session.

## Workflow General Instructions
- Always use --headed browser mode.
- after executing commands wait for browser to be stable and do not hurry to execute next command.
- Do not click/input anywhere else on the page, only click on the elements which are required to be clicked/input as per the workflow.
- Always take screenshot after each step and save it in the current project root directory with a unique name.
