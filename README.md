# Aws-workshop-template

## Repo structure

```bash
.
├── contentspec.yaml                  <-- Specifies the version of the content
├── README.md                         <-- This instructions file
├── deck                              <-- Directory for presentation deck
├── static                            <-- Directory for static assets to be hosted alongside the workshop (ie. images, scripts, documents, etc) 
├── resources                         <-- Directory for workshop resources
│   ├── code-samples                  <-- Directory for workshop modules code
│   └── cfn-templates                 <-- Directory for workshop modules CloudFormation templates
└── content                           <-- Directory for workshop content markdown
    └── index.en.md                   <-- At the root of each directory, there must be at least one markdown file
    └── introduction                  <-- Directory for workshop content markdown
        └── index.en.md               <-- Markdown file that would be render 
```

## What's Included

This project the following folders:

* `deck`: The folder to store your presentation materials.
* `resources`:  Store any example code, IAM policies, or Cloudformation templates needed by your workshop here.
* `content`: This is the core workshop folder. This is generated as HTML and hosted for presentation for customers.

## How to create content

Under the `content` folder, Each folder requires at least one `index.<lang>.md` file. The file will have a header

```aidl
+++
title = "AWS Workshop Template"
weight = 0
+++
```

The title will be the title on navigation panel on the left. The weight determines the order the page appears in the navigation panel.