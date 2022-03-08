# ArgoCD Exercise

## Prereqs
Make sure you have gone through the Kubernetes cluster setup prior to this guide.
The docs for ArgoCD: https://argo-cd.readthedocs.io/en/stable/

## Install ArgoCD
```
kubectl create namespace argocd
kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Accessing ArgoCD UI

#### Note: While you don't need to view the ArgoCD UI it does look pretty cool 


In order to log in to the ArgoCD UI, run the following command

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d | pbcopy
```

The previous command retrieved the generated password that will be used to log in to the UI with the user "admin" and the password was copied into your clipboard (courtesy of `pbcopy`)

In order to view the ArgoCD UI, run the following command:

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

This command allows us to view the ArgoCD UI by visiting `http://localhost:8080` in our browswer. 
### Note: Your browser might complain about it being an "insecure" site, but this is because we have not configured the UI with proper SSL Certificates so proceed since we are not going to steal our own data

The UI should look like the below (at least on March 1st, 2022, it did)
![login image](/media/login.png)

Enter "admin" as the username and use what's in your clipboard (or run that command again) as the password


After logging in you the Applications Dashboard should look like the below 
The UI should look like the below (at least on March 1st, 2022, it did)
![ArgoCD NoApps](/media/argocd-dashboard-no-apps.png)

Wow so much nothing! Move on to the deploy step so we can actually see something interesting on this page

## Deploying an application

 In the guestbook-api-config directory, you will see two files. A `deployment.yaml` and a `service.yaml`, this is the end result of what will be created on Kubernetes. ArgoCD will do this by providing an `Application` CRD. The Application CRD is the Kubernetes resource object representing a deployed application instance in an environment. It is defined by two key pieces of information: a `source` and a `destination`. 

 See [guestbook-application.yaml](/guestbook-application.yaml) and read more: [ArgoCD Application](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#applications)


Now that you understand what an `Application` is in ArgoCD terms, let's create one!

```
kubectl apply -f guestbook-application.yaml
```

Expected output
```
application.argoproj.io/guestbook created
```

So....guess what? Your app is up and running.

Don't believe me, check our your ArgoCD Dashboard

![ArgoCD Dashboard Apps](/media/argocd-dashboard-apps.png)

After clicking on the app in the dashboard, you can see what components that are getting created:

![ArgoCD Dashboard Apps](/media/argocd-dashboard-guestbook-pipeline.png)

And finally, you can run one more command and see that the application is running as expected

```
kubectl port-forward svc/guestbook-ui -n guestbook 8000:80
```

Now visit `http://localhost:8000` to see the guestbook app

![ArgoCD Dashboard Apps](/media/argocd-guestbook-app.png)

#### So in summation, ArgoCD is really cool and this is a very basic exercise. There is more to learn and much more to do


## Deploying an application of applications (app of apps)
Deploying an application of applications (app of apps)

Deploying a single app is cool, but who runs just one app anymore? 

ArgoCD actually allows us to deploy multiple apps by defining a single top level app. This paradigm is called "app of apps". App Of Apps

If you run the following command (that references the attached app-of-apps.application.yaml file)
```
kubectl apply -f app-of-apps.application.yaml
```

Expected output
```
application.argoproj.io/guestbook-apps created
```

Now if you visit the ArgoCD UI you will see that multiple apps were created

![ArgoCD Dashboard App of Apps](/media/argocd-dashboard-app-of-apps.png)

If you click on guestbook-apps, you can see the relationship of how this single app created the tree of other apps
![ArgoCD Dashboard App of Apps](/media/argocd-guestbook-app-of-apps.png)

Cleaning up the application can be done by running kubectl delete
```
kubectl delete -f guestbook-application.yaml
```

Coming soon

## Yo dog, I heard you want ArgoCD so I ArgoCD'd your ArgoCD
https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#manage-argo-cd-using-argo-cd
