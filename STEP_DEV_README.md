# Step Development

---

This guide will teach you how to write your own bitrise step.
In this guide we will create a step which will:

- read your ios project's version (CFBundleShortVersionString) and assign to variable
- optionaly add prefix to the version

## Getting started

---

Best practice is first creating a script file with your functionality.

Our script looks like:

```
#!/bin/bash

set -e

PLIST_PATH="path/to/my/project/Info.plist"
version=$(/usr/libexec/PlistBuddy -c "Print CFBundleShortVersionString" $PLIST_PATH)
echo "version: $version"

PREFIX="release version:"
prefixed_version="$PREFIX $version"
echo "prefixed_version: $prefixed_version"

```

This is a bash script which uses `PlistBuddy` to read the `CFBundleShortVersionString` from `$plist_path` and prints the version.

Then the script appends `release version:` prefix to the version and prints the prefixed_version.

In my case the output is:

```
version: 0.9.3
prefixed_version: release version: 0.9.3
```

## Try out the script on bitrise.io

---

It is recommanded to validate that all of our script's requirements are available on bitrise machines.

To do that add a `script` step to your workflow on bitrise.io.

In this case my `bitrise.yml` looks like:

```
...
    - script@1.1.0:
        inputs:
        - content: |-
            #!/bin/bash

            set -e

			plist_path="path/to/my/project/Info.plist"
			version=$(/usr/libexec/PlistBuddy -c "Print CFBundleShortVersionString" $plist_path)
			echo "version: $version"

			prefix="release version:"
			prefixed_version="$prefix $version"
			echo "prefixed_version: $prefixed_version"
...
```

If your build succeed and you are happy with the logs, you validated that all of the step's requirements are available on bitrise.io.

## Convert the script into a step

---

To fulfill this stage do the followings:

**1. create your step's git repository and clone it**

**2. copy the step template files (https://github.com/bitrise-steplib/step-template) into your repository**

**3. fill the step.sh with your script**

**4. wire out your inputs to step.yml (`inputs` field)**

In our case the step has an required input (`plist_path`) and an optiona input (`prefix`).

So the step.yml should looks like:

```
...
run_if: ""
inputs:
- plist_path:
  opts:
   title: "Info.plist path"
   summary: Info.plist path
   description: |
     Path to your Info.plist.
   is_expand: true
   is_required: true
   is_dont_change_value: false
   value_options: []
- prefix:
  opts:
   title: "Version prefix"
   summary: Version prefix
   description: |
     This prefix will be append to your version.
   is_expand: true
   is_required: false
   is_dont_change_value: false
   value_options: []
outputs:
...
```

Notes:

- `is_expand`: if true the provided input value will be expanded, otherwise not.

  (example: if true $HOME will be the value of your $HOME env)

- `is_required`: set it true if input is required

- `is_dont_change_value`: if true you can not change this input's value on bitrise.io graphical workflow editor.

- `value_options`: if there is a set of available values, list that there and in this case you should provide default value for the input as well.


**5. fill the step.yml other fields**

**6. open bitrise.yml and do the following changes:**


```
...
workflows:
  test:
...
    - path::./:
        title: My version exporter step
        inputs:
        - plist_path: path/to/my/project/Info.plist
        - prefix: "My version"
```

Now bitrise will read the step's inputs (`plist_path`, `prefix`) and pass those to the step as environment variable.

To use the provided inputs, we have to change the `step.sh`.

```
#!/bin/bash

echo "Configs:"
echo "plist_path: $plist_path"
echo "prefix: $prefix"
echo

version=$(/usr/libexec/PlistBuddy -c "Print CFBundleShortVersionString" $plist_path)
echo "version: $version"

prefixed_version="$prefix $version"
echo "prefixed_version: $prefixed_version"
```

Now you can test the step by calling `bitrise run test`.

In the log you should find your step's log, something like this:

```
...
Configs:
plist_path: path/to/my/project/Info.plist
prefix: My version

version: 0.9.3
prefixed_version: My version 0.9.3
...
```

At this point your step is ready to use.

To try out your step, push the changes to the repository.

On bitrise.io in your bitrise.yml you can use your git step:

```
...
workflows:
  primary:
...
    - git::https://github.com/user/my-step.git@master:
        title: My version exporter step
        inputs:
        - plist_path: path/to/my/project/Info.plist
        - prefix: "My version"
...
```
