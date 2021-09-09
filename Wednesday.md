# Wednesday
## Topic:
    - Getting SSH keys configured
    - CICD
    - Jenkins
    - webhooks 
     
Getting SSH keys to generate:

Locate to the ssh location:  `cd ~/.ssh`
```
Type this: ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
call the key e.g. `sre_jenkins`
```
## In Github in side your selected repo
- Go into github setting and select the `GCP and SSH settings` 

- Add a SSH key copy the generated public key from your *Bash* file into the box in github
- ###  Make sure to allow **Write** access  (its at the bottom) 

## In Bash: Now you are gonna connect to the repo 
-  first clone the repor into your directory

 ```
 Git clone [repo link]
 git init (inside the downloaded repo)
 git remote add origin [SSH link]
 Now you are connected to the repo
```
## END
___
## Jenkins: Webhooks 
` This means if there is a change in your repo then jenkins will 
catch it and do somthing about it
`

### The Testing Jenkins part 1:
#### This Jenkins basically Gets the fies from the main repo and runs test in an instance

## Configuring the Item

- Access your Jenkins and create a new items
- Name the *new item* e.g. SRE_CI
- Select the freestyle project and click `ok` at the bottom
- select the first option and click continue 

Set the configuration as described below 
```
1: Description: anything you want
2: Discard old builds = ticked 
    -Strategy
        - log Rotataion
    - Max # of builds to keep
        - 3
3: GitHub project = ticked
    -Project url
        - your project url

4: Restrict where this project can be run =ticked
    - Select the server you want to run the test on e.g. `sparta-ubuntu-node`

5: Git = ticked
    Repositories:
        - Repository URL = the github url
        - Credentials = jenkins key (the one you linked to github) 
    Branches to build:    
        - Branch Specifier (blank for 'any') = */main
    
6: GitHub hook trigger for GITScm polling = ticked

7: Provide Node & npm bin/ folder to PATH = ticked 
    - The values will all be Auto assigned 

8: Slect `Add build step` in the Build section:
    - Execute shell
    - Type this in the commands box:
        cd app
        npm install 
        npm test   
        
9: Add post-build action:
    - Projects to build [Merge item] (Not created yet)
click SAVE    
```
### Once you save somthing to your repo it gets tested in the instance and you will recive the results in the **Console log**

___


## The Merging Jenkins part 2:
### This Jenkins merges your choosen branch to the main branch 


Follow steps 1-5 from the previous section 

- Get to `Branches to build`:
    - change the bank box to your choosen branch e.g. `*/dev`
    - Additional Behaviours:
    ```
     Name of repository: origin
     Branch to merge to: main
     Merge strategy: default
     Fast-forward mode: --ff
    ```
follow steps 6-7

- Add post-build action:
    - Projects to build: [The previous items name e.g. SRE_item]
    - Git Publisher:
    ```
    Push Only If Build Succeeds =ticked
    Merge Results =ticked

    ```
## Apply and save 
## END 
___


##  Create New item 
### The Merging Jenkins part 3:
 ```
 This Jenkins item pushes the repo changes to the AWS instance and runs it
 ```

follow the first  1-6 (Skipping only step 4)  from the first jenkins installation:
- Build Environment:
    -SSH Agent:
        - Credentials: set this key to the AWS prem key, 
        this is done by click add and following instructions 

Build:
Add buiild step: 
    -   Execute shell:
```
ssh -A -o "StrictHostKeyChecking=no" ubuntu@[AWS SSH connection ]<< EOF
git clone https://github.com/om1chael/Jenkins_APP.git
cd Jenkins_APP
export DB_HOST=[DB private ip]]/posts/
git pull 
cd app
pkill -f node
npm start 
EOF

```

END