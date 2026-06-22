### Failure 1: Dependency Install Step

- Step Name: Install Dependencies
- Error Message:
  "npm ci can only install packages when your package.json and package-lock.json are in sync"
  "Missing: lodash@4.18.1 from lock file"

- Explanation:
  The package.json file includes lodash@4.18.1, but the package-lock.json file is out of sync and does not include this dependency. Since npm ci requires an exact match between the two files, the install step fails.