# — Second Secrets and variables.
So in this we are going to see how we can define variables and secrets in github actions we will use them in future 
while implementing pipeline.

go in your repositry then settings there you'll see environments and  secrets,  variables. 

click on it Secrets and variables.

actions:

Secrets:

in actions you'll see  store New repositry secrets for your repo pipeline you can use them later in the pipeline 
For example you can store here  passwords or registry credentials, tokens etc and the data will be encrypted. 
so basically in secrets you can store all the confidential data.

Variables:

there are also variables you can store variables and use them in variables we generally store data which is not 
confidential. If we want to change the value again and again so we store it in 1 place and only change the value 
there and then in the pipeline its value is automatically updated every where.

Environment: 

Before secrets and variables we have environment it can be prod, dev , test, Qa, staging etc. so you can set different 
variables and environments for example you give a variable and its value is something and you have 2 environments you 
can have a same variable but value for both environments can be different if you set variables or secrets in an environment 
its a best practice to avoid conflicts while having many environments in production server the pipeline will have multiple 
environments for example testing development etc etc.

Create a environment: For example => Production

click on manage environemnt secret:

now you can add sceret and variables: youre going to see if you add a secret and access it again the value will not be shown 
you can just update it with a new value. but in variables you can see the value right there. secrets and variables are highly 
used in a production workflow for storing aws creds, sonar qube server url, docker creds, ecs creds, and many more things.
