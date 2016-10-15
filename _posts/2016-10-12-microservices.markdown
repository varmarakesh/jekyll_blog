---
layout: post
title:  "Micro-services"
date:   2016-10-12 15:34:11
categories: services
comments: True
---
Micro-services design is really helping alleviate some problems we experienced with the monolithic model. Implementing micro-services is perhaps one of the greatest ways to improve the producitivity of a software engineering team.

Specifically,

*	You deploy only the component that was changed (as opposed to the whole application), this keeps the deployments and the tests managable.
*	If multiple team members are working on the application, they need to wait until everyone is done with development, testing and ready to move forward. However, with microservices-services model, whenever each piece of functionality is ready it could be deployed.
*	Each micro-service runs in its own process space, so if we want to scale for any reason, we can scale the particular micro-service that needs more resources instead of 	scaling the entire application.
*	When one micro-service needs to communicate with other other micro-service, it does using a lightweight protocol such as http.

This post examines, how to deploy microservices created using python and django on nginx and uwsgi.

Before going into the details of a microservices design, lets understand and examine how a typical monolithic application looks like.

##### Monolithic Design
![monolithic-design](/assets/images/monolithic_service_design.png){:class="img-responsive"}

However, with breaking the functionality into logically isolated micro-services, this is how the design would like.

##### Microservices Design
![microservices-design](/assets/images/microservice_design.png){:class="img-responsive"}

Now, lets examine how the configuration would like for a python and django application that runs on nginx on a typical linux server.

Because the application code is spread across multiple repos (or subrepos) grouped by logically independent code, this will be the typical organization of the application directories on the server.

{% highlight bash %}
[ec2-user@ip-172-31-34-107 www]$ pwd
/opt/www
[ec2-user@ip-172-31-34-107 www]$ ls -lrt
total 8
drwxr-xr-x. 5 root root 4096 Oct 12 14:09 microservice1
drwxr-xr-x. 7 root root 4096 Oct 12 19:00 microservice2
drwxr-xr-x. 5 root root 4096 Oct 12 14:09 microservice3
drwxr-xr-x. 7 root root 4096 Oct 12 19:00 microservice4
{% endhighlight %}

Nginx, which is typically deployed a front-end gateway or a reverse proxy will have the this configuration.

{% highlight bash %}
[ec2-user@ip-172-31-34-107 serviceA]$ cat /etc/nginx/conf.d/service.conf 
upstream django1 {
    server unix:///opt/www/service1/uwsgi.sock; # for a file socket
}

upstream django2 {
    server unix:///opt/www/service2/uwsgi.sock; # for a file socket
}

upstream django3 {
    server unix:///opt/www/service3/uwsgi.sock; # for a file socket
}

upstream django4 {
    server unix:///opt/www/service4/uwsgi.sock; # for a file socket
}

server {
    # the port your site will be served on
    listen      80;
    # the domain name it will serve for
    server_name localhost;
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    location /api/service1/ {
	    uwsgi_pass  django1; 
	    include     /etc/nginx/uwsgi_params; 
    }
 
   location /api/service2/ {
            uwsgi_pass  django2;
            include     /etc/nginx/uwsgi_params; 
    }

    location /api/service3/ {
	    uwsgi_pass  django3; 
	    include     /etc/nginx/uwsgi_params; 
    }
 
   location /api/service4/ {
            uwsgi_pass  django4;
            include     /etc/nginx/uwsgi_params; 
}
{% endhighlight %}

Multiple uWSGI process have to be created that would process the request for each microservice.

{% highlight bash %}
/usr/bin/uwsgi --socket=/opt/www/service1/uwsgi.sock --module=microservice-test.wsgi --master=true --chdir=/opt/www/service1

/usr/bin/uwsgi --socket=/opt/www/service2/uwsgi.sock --module=microservice-test.wsgi --master=true --chdir=/opt/www/service2

/usr/bin/uwsgi --socket=/opt/www/service3/uwsgi.sock --module=microservice-test.wsgi --master=true --chdir=/opt/www/service3

/usr/bin/uwsgi --socket=/opt/www/service4/uwsgi.sock --module=microservice-test.wsgi --master=true --chdir=/opt/www/service4
{% endhighlight %}
