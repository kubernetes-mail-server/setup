# Kubernetes Mail Server

## Cluster Setup

The cluster requires a namespace to install all the projects into, as well as a service account for deploying the software through your pipeline. Each project contains the required steps to deploy usng a Travis-CI and GitLab pipeline.

The gitlab pipeline does not support encrypted variables. Therefore you should add deploy keys and environment variables through the project administration inside GitLab. See the GitLab configuration section for help with this.

Things you will need to change:
- **TODO**: write a list of things that the user has to edit 

Things you will need installed:
- **MacOS**: You should install brew, if you don't want to, you're on your own regarding those tools
- A kubernetes cluster somewhere
- A working kubectl for that cluster
- A working helm tool installed to apply this projects 'setup' configuration
- If using travis, please install the travis gem for your operating system
- The create_kubeconfig script will complain if you don't have yq installed so it can easily parse the output of the values.yaml file

To install "kubectl" (You didn't install this yet? ok!):
- **MacOS**: "brew install kubectl"
- **Windows**: (TODO) 

To install "yq":
- **MacOS**: brew install yq
- **Windows**: (TODO)

## Generating a kubeconfig file to use for deployment

Just run the ```create_kubeconfig``` script from this project. It will read the values.yaml file and generate what should be a working kubeconfig. 
**It might be that this tool isn't perfect.** 

```
cd setup
./create_kubeconfig
```


To test it, do the following

```
export KUBECONFIG=$PWD/kubeconfig

# This should reply 'pod/hello-world created'
kubectl run hello-world --generator=run-pod/v1 --image hello-world

# Look at the logs to see whether it's running as expected
kubectl logs -f hello-world

# When you're done, delete it
kubectl delete pod hello-world

# It should be gone
kubectl get pods 
```

## Configuration with Gitlab CI

Firstly Take the kubeconfig generated and pass it through base64
```
cat kubeconfig | base64
```

Copy that output and paste it into an environment variable on your project called **CI_KUBECONFIG**

Environment variables for each project are accessible in the **Settings > CI/CD > Variables** section 

You will need **Deploy Keys** for each project. Access them in the **Settings > Repository** section

Create a new deploy key, call it **kubernetes** (this name isn't important, it's just so you can recognise it later). It will give you a username and password. 

Create two new environment variables for your project. Call them **CI_DEPLOY_USERNAME** and **CI_DEPLOY_PASSWORD** and copy/paste the appropriate values

**NOTE:** You aren't forced to use these names, but if you do change them, you're responsible for making sure everything else is changed to match with the names you wanted to use.

## Configuration with Travis-CI

If using the travis pipeline, each project requires a kubeconfig file to be supplied and encrypted to decrypt inside each stage of the pipeline and give you access to your cluster.

- Copy the kubeconfig into the project you want to configure

**NOTE: You must add kubeconfig file to your .gitignore. If you don't, you're potentially going to make a mistake and commit+push your unencrypted kubeconfig to your publically available project. You'll get fucked in ways you thought were just in comic books :)**

- Run: ```echo "kubeconfig" >> .gitignore```
- Run: ```travis encrypt-file --com kubeconfig```
- It will give you an openssl command to add to your .travis.yml file
```
# Example: 
before_install:
  - openssl aes-256-cbc -K <snip> -iv <snip> -in kubeconfig.enc -out kubeconfig -d
```
- Run: ```git add kubeconfig.enc .gitignore```
- Run: ```git commit -m "added encrypted kubeconfig" && git push```