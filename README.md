# AWS Load Balanced Website Project
​​
## Project Overview
​
**Amazon Web Services (AWS)** is the world leading cloud service provide with a massive global footprint spans over 102 Availability Zones and 32 geographic regions (4 more on the way including Canada West, New Zealand and Thailand). As the world moves on to rely more and more on cloud computing, an in-depth understand of cloud environment is a must for every inspiring cybersecurity personnel.
​<br>

**Amazon Lightsail** is an easy way for developers to access compute, storage and networking capacity and capabilities of AWS to build applications and websites with low-cost in a pre-configured package.
​​<br>

This is a website I've made for myself as a simple AWS showcase project. This is not a step-by-step tutorial. I didn't document my workflow when I was building the website. But I will describe the general process on how I created the website using AWS Lightsail. 
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/aws-logo.png)
​​<br>
<br>
## Project Architecture
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/AWS_Lightsail_Project.drawio.jpg)
​​<br>
<br>
My website project process:
​
1. Create primary WordPress instance
​
2. Attach Static IP to the WordPress instance
​
3. Create Lightsail Object Storage
​
4. Install S3 Storage Offload Plugin for WordPress
​
5. Complete the WordPress website design
​
6. Create Lightsail Database
​
7. Connect the Primary WordPress instance to the Lightsail Database
​
8. Create a snapshot of the primary WordPress instance
​
9. Create a secondary WordPress instance from the snapshot
​
10. Attach the Lightsail Object Storage to the secondary WordPress instance
​
11. Purchase and activate your website domain using AWS Route 53
​
12. Create Lightsail Load Balancer
​
13. Enable CloudFront CDN distribution on the Lightsail Object Storage
​
## Project Process
​
### Create Primary WordPress Instance
​
First, we will need to create a Lightsail WordPress instance. Be careful when picking your region and availability zone because some of the services may not be available/accessible in that region/availability zone.
​
This WordPress instance will serve as our primary server. We will be taking a snapshot of it and use the snapshot to spin up an identical WordPress server in order to utilize the load balancer feature later.
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/1-Create-WordPress-Instance.png)
​​<br>

![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/1-Create-WordPress-Instance-2.png)
​​<br>
<br>
### Attach Static IP to the WordPress Instance
​
In order to connect our instances later to the load balancer, each instance must have its own static IP address. Pick the same availability zone as your WordPress instance. Once its created, attach it to the WordPress instance.
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/2-Adding%20Static%20IP.png)
​​<br>
<br>
### Create Lightsail Object Storage
​
Each WordPress instance will have its own SSD for storage. However, we need a common storage for all of our instances to access. To do that, we need to create a Lightsail Object Storage. And since the Lightsail Object Storage is really just AWS S3 buckets, we can take advantage of this by connecting it to CloudFront for fast global content delivery.
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/3-Create-Object-Storage.png)
​​<br>
<br>
### Install S3 Offload Plugin for WordPress
​
Now we need to allow our WordPress instance access to our Lightsail storage bucket
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/4-S3-offload-1.png)
​​<br>

![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/4-S3-offload-2.png)
​​<br>
<br>
### Complete Website Design
​
Now we can finish our website design at this step. The reason we need to complete the construction of our website here is because we will be creating a Lightsail database to share between our WordPress instances. It's best to set up everything here then copy the MySQL database over to ensure nothing is going to be missing.
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/Simon-homepage.png)
​​<br>
<br>
### Create Lightsail Database
​
Each WordPress instance has its own MySQL database. However, in order to use the load balance feature, our instances must have a shared database. So we need to create a Lightsail Database and use it as the shared database for our instances.

Use the below command to create a backup copy of your WordPress database.
```bash 
sudo mysqldump -u root -p bitnami_wordpress > YOUR_DATABASE_BACKUP_NAME
``` 

Then use the below command to copy your current WordPress database into the new Lightsail database
```bash
sudo mysql -h YOUR_LIGHTSAIL_DATABASE_ENDPOINT -u LIGHTSAIL_DATABASE_USERNAME -p LIGHTSAIL_DATABASE_NAME < YOUR_DATABASE_BACKUP_NAME
```
<br>

![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/6-Create-Database-1.png)
​​<br>

![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/6-Create-Database-2.png)
​​<br>

![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/6-Create-Database-3.png)
​​<br>
<br>
### Connect WordPress to the Lightsail Database
​
We've successfully copied the existing MySQL contents from our WordPress instance to the shared Lightsail Database. However, we need to connect the WordPress instance to the Lightsail Database so that it could start writing all the new changes and contents into the Lightsail Database instead of its own. Fortunately, all we have to do is to change the default database values inside the `wp-config.php` file.
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/7-Connect-Lightsail-Database.png)
​​<br>
<br>
### Create a snapshot of the primary WordPress instance
​
A snapshot is just a backup copy of your system at the very moment in time: every file, every configuration is preserved exactly the way they are. I took a snapshot of the primary WordPress instance right after it was created and after I've finished designing it. The reason I took a snapshot right after the instance was first created was because I wanted to ensure that if I somehow broke the WordPress installation due to customization, I could easily use the snapshot to disregard all the changes and start from a clean slate again.
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/8-Primary-wp-snapshot.png)
​​<br>
<br>
### Create a secondary WordPress instance from the snapshot
​
Now we can create an identical WordPress instance from the snapshot we just took. Additionally, don't forget to create and attach a new static IP to this instance.
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/9-Create-instance-from-snapshot.png)
​​<br>
<br>
### Attach the Lightsail Object Storage to the secondary WordPress instance
​
Just like we did for the primary WordPress instance, we need to attach this instance to the object storage as well.
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/10-Object-storage-second-wp.png)
​​<br>
<br>
### Purchase and activate your website domain using AWS Route 53
​
Now that we have both identical WordPress instances with shared Lightsail Database up and running, the next step is to purchase a domain so we point our website to it. In this project here, we are using [simonjiang.net](https://www.simonjiang.net) as our project domain name. You can register the domain and configure the DNS records through AWS Route 53. You will need to head back to Lightsail's own "Domain & DNS" page to complete the DNS configuration.
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/11-Domain-1.png)
​​<br>

![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/11-Domain-2.png)
​​<br>
<br>
### Create Lightsail Load Balancer
​
Now that we have properly set up the domain and we have both instances of WordPress up and running, it's time to connect them to the Lightsail Load Balancer. The Load Balancer will automatically forward the visitor traffic to either one of our WordPress servers based on server usage. Make sure to turn on the "Session persistence". This will allow the visitor to stay with the instance they first connected with for consistency. Here's also where we will get our SSL Certificate to be signed and authenticated so that we can have HTTPS traffic enabled on our website.
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/12-Load-Balancer.png)
​​<br>

![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/12-Load-Balancer2.png)
​​<br>
<br>
### Enable CloudFront CDN distribution on the Lightsail Object Storage
​
Our last step is to set up the CloudFront CDN distribution for our WordPress website. This will allow all of our static contents to be served to global edge locations for fastest content delivery. All we need to do here is to create a CDN distribution and point it to our object bucket where all of our static contents are stored. We can also get our SSL Certificate to be signed and authenticated here.
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/13-CloudFront-CDN.png)
​​<br>

![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/13-CloudFront-CDN2.png)
​​<br>
<br>
### Final Result
​
After all that work, now we can finally enjoy our shiny new website. Looks very groovy! You can **[check out the project website here](https://www.simonjiang.com)**.
​​<br>
<br>
![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/Simon-homepage.png)
​<br>

![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/simon-about.png)
​<br>

![](https://github.com/thesimonjiang/AWS-Lightsail-WordPress-Project/blob/71dd04da3c4c34cc4b36efa38b97f79785d8aa47/Graphics/simon-portfolio-page.png)
