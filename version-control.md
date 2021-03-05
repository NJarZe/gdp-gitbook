# Version Control & Application Lifecycle Management
Version control, also known as source control, is the practice of tracking and managing changes to software code. Version control systems are software tools that help software teams manage changes to source code over time. As development environments have accelerated, version control systems help software teams work faster and smarter.

---

Application lifecycle management (ALM) is the people, tools, and processes that manage the life cycle of an application from conception to end of life. It is made up of several disciplines that have often been separated under legacy development processes, such as a waterfall development method, including project management, requirements management, software development, testing and quality assurance, deployment, and maintenance.

Application lifecycle management supports **agile** and **DevOps** development approaches by integrating these disciplines together and enabling teams to collaborate more effectively. Adopting ALM also leads to **continuous delivery** of software and updates with frequent releases, sometimes as often as several per day, as opposed to new releases only coming every few months or once a year.

## Version Control
For all Global Data Platform projects, [Azure DevOps](https://dev.azure.com/) is the tool that will be used host and track all code that is written, including Azure Data Factory pipelines and artifacts. 

**Git** will be used as the version control system, and **Agile** will be used as the Work item process for each project. 

## Azure DevOps Organization
Within Azure DevOps there is a DevOps Organization called "AliaxisDataPlatform". This organization will host all the (DevOps) projects that will contain the code for each project related to Global Data Platform. 

Each (DevOps) Project in the AliaxisDataPlatform organization represents a division. That is, all the code and artifacts for a given division can be found under the project with the division's name (for example, all EMEA projects, including Snowflake code and Data Factory artifacts, will be hosted in a DevOps Project called **EMEA**).

For every project that is being worked on within a division, there are several components that need to exist in Azure DevOps
1. A Team within the Project: This will manage who can access which code repositories.
2. A Repository: All project-related code will be saved in this repository (Snowflake code, Data Factory artifacts, anything else that needs to be versioned and tracked.).

Within each repository (which represents a data platform project), folders will be used to separate the different types of artifacts that are stored in the repo. 
For example, if there is a new project for "Group Contribution" in the EMEA division, which will be worked on the Data Platform, the following must be done. 
1. Create a new repo in the EMEA DevOps Project.
2. Create a new folder in the repo called "data_factory" for all Data Factory artifacts.
3. Create a new folder in the repo called "SnowflakeScripts"
4. Create subfolders under "SnowflakeScripts" to further organize the code. See Naming Conventions and Folder Organization for further reference.

## Ownership and Governance
Since each DevOps project represents a Division, the ownership of each DevOps project will be the Division's data platform lead. This is the person that is managing or leading data-platform projects in that division, and whom is aware of ongoing and upcoming activities (i.e. the Division's backlog).

When a repo is created, due to a new data-platform project, the owner of the repo is the project lead. The project lead (and repo owner) is in charge of maintaining the repo's folder structure and organization, making sure the guidelines are being followed for all artifacts and code. 

The developers that are actively working on a data-platform project will have access to the repository(ies) that are relevant for them. These developers are the ones actively contributing to the project with code and/or other artifacts. 

## Naming Conventions and Folder Organization
### DevOps Projects (Divisions)
Divisions will be named with the division acronym, as follows.
- EMEA: Will contain projects for the EMEA division.
- NA: Will contain projects for the North American division.
- APAC: Will contain projects for Asia Pacific.
- LATAM: Will contain projects for Latin America.
- INDIA: Will contain projects for India.
- GLOBAL: Will contain projects for Group (HQ). These projects are usually managed and worked on by the BICC. 

### DevOps Repositories (Data Platform Projects)
Each repository must be named after the project it represents. 
The name must be in **lower case** letters.
If there are multiple words in the project name, the words must be separated by a **hyphen (-)**.

For example,
|Project name        |Git Repo Name       |
|--------------------|--------------------|
|End To End Margin   |end-to-end-margin   |
|Resin Spend Analysis|resin-spend-analysis|

### Folders within a Repository
Every repo should have at least two high/root level folders: data-factory and snowflake-scripts
- data-factory: This folder is where data factory will store its artifacts and code.
- snowflake-scripts: This folder will contain all the Snowflake code.

Within the snowflake-scripts folder, there should be at least 2 subfolders: dimensions, and facts. Within each subfolder, there should be the following folders: 
- tables: Contains scripts for table creation, alteration, etc. (DDL - data definition language). 
- stored-procedures: Contains scripts for stored procedure creation.
- views: Contains scripts for view creation.
- other: Contains all other scripts.

## Git Repositories (repos)
As mentioned in Azure DevOps Organization, each Git repository (repo) represents a data-platform project. 
Repos for projects that have been closed and/or decommissioned will remain visible for historical and documentation purposes. 
When a new data-platform project needs to be worked on, the owner of the DevOps Project (Division Data Platform lead) must create a new repository that will contain the code for the project (see Naming Conventions and Folder Organization). 

Since each repository is specific to a data-platform project, only the team that is actively working on the project will have access to contribute to the repo. If other developers need access to certain code, they can request read-only access to the repo; the project lead will determine whether to grant access to the repo or not. 

## Branching Strategy
Git offers the ability to create branches for different purposes. 
Highlighted below are the guidelines for default (mandatory) branches, and branches that can be created on an as-needed basis, or temporary branches. 

### Default (mandatory) branches
Every repo **must** have at least **two** branches that will live permanently in the repo (i.e. they should never be deleted): master and dev. 
- Master branch (name: master): This is the default branch for every new repo. It should always contain stable code and **should not** allow direct check-ins or commits. The only way to get code into the master branch is via a Pull Request with code review (see Merging Strategy). 
- Dev branch (name: dev): This is the main development branch. Developers can use this as a working branch and will make changes to it regularly. 

### Temporary branches
These branches are created by developers when they need to work on a specific request or bug fix, which is not part of the ongoing development. 
Some sample scenarios in which developers will need to create temporary branches: 
- Bug fix
- New feature (which was not originally part of the roadmap/backlog)
- Experiments (for testing or experimenting)
- Personal WIP (work in progress) branches
- New release

When a temporary branch is merged into another branch, the temporary branch should be deleted (hence temporary).

### Who should create branches? 
The default branches should be created when the repo is created, by the owner of the repo or the owner of the DevOps Project (Division lead). 

Temporary branches can be created by developers or repo owner as needed. 

### The "Release" branch
Every time a new release is ready to be sent to production, a new release branch must be created. The release branch is a temporary branch that is identified as a release branch by its name (see Branch naming convention). 

When a new release branch is created, the release version should be added to the changelog (see Changelog) along with a description of what is included in the release. This will help keep track of when specific features or enhancements were introduced.

### Branch naming convention
All branch names must be **lower case** and should describe the purpose of the branch. If the branch name contains several words, words must be separated by **hyphen (-)**.

For default branches, the name must always be **master** and **dev**. 
Release branches must start with the word **release** followed by **hyphen** and the release number in the X.Y.Z format (major.minor.patch).

Examples for temporary branches
- wip-new-margin-calculation
- bug-net-price-issue
- feature-adding-data-quality-checks
- release-1.0.2

## Commits
Commits are allowed, by default, on any branch except master. 
Only users that are contributing to the project are allowed to commit to the branches in the repo. 

All commits go to the branch currently being worked on. Code that needs to be moved between branches (merged), will be moved based on the merging strategy.

## Merging Strategy
Git allows for several ways to merge code from one branch into another branch, direct merge and pull reqeust. 

### Direct merge
Direct merge means a developer, or the repo owner, can merge the code from one branch to another with a simple git command. This method should only be used when merging into any branch other than master. 

Note: If a temporary branch is merged into another branch, the temporary branch should be deleted. 

### Pull Request
This method must **always** be used when merging code into the master branch. 

When code has been tested by the developer, tested in the QA environment, and ready to be sent to production, a new "release branch" should be created as defined in "The Release Branch". 

Once ready to merge this code into the master branch, a **Pull Request** must be created via the DevOps user interface. The Pull Request should include approvers, who will review the changes and approve/decline as they deem convenient. When all approvals are done, the pull request is ready to be executed. Executing the pull request merges the code into the master branch and deletes the release branch. 


## Changelog
At the root of the repo, there should always be a filed called Changelog.md. This file will contain every release that has been sent to production, along with a description of what is included in each release (bugs fixed, known issues, new features, etc.).

Every entry in the changelog must include
- Date
- Release version (major.minor.patch)
- Notes

The changelog entries should be kept in reverse chronological order (newest to oldest); that way, the most recent release is always the first entry in the file.