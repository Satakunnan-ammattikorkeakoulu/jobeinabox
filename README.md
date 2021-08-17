# JobeInABox customized for SAMK

This is a fork of [trampgeek/JobeInABox](https://github.com/trampgeek/jobeinabox) that introduces additional python 3 libraries for AI.

This readme contains instructions for setting up [CodeRunner](https://coderunner.org.nz/) Moodle plugin and the required [Jobe](https://github.com/trampgeek/jobe) server using this repository as the Jobe image. This step-by-step tutorial has been made to work with Jobe server running on Ubuntu 20.04.

# Introducing the Products

In this tutorial we will install both CodeRunner Moodle plugin and Jobe server.

## CodeRunner

CodeRunner is a free open-source question-type plug-in for Moodle that can run program code submitted by students in answer to a wide range of programming questions in many different languages. It is intended primarily for use in computer programming courses although it can be used to grade any question for which the answer is text. ([source](https://coderunner.org.nz/))

Official documentation can be found [here](https://github.com/trampgeek/moodle-qtype_coderunner#code-runner).

## Jobe Server

Jobe (short for Job Engine) is a server that supports running of small compile-and-run jobs in a variety of programming languages. It was developed as a remote sandbox for use by CodeRunner, a Moodle question-type plugin that asks students to write code to some relatively simple specification. However, Jobe servers could be useful in a variety of other contexts, particularly in education. ([source](https://github.com/trampgeek/jobe))

# Installation

## Prerequirements

Before starting the installation, you need to first:

1. Have Moodle 3.x server running and have administrator rights to that Moodle server
2. Have a server ready where we can setup the Jobe server. The most optiomal solution would be to have the Jobe server installed in the same network as the Moodle server to get best security and performance. This server can be any Unix server that can run Docker like Ubuntu or CentOS.

## Steps

Installation is fairly straightforward:

1. Install Jobe in it's own Ubuntu 20.04 server
2. Install the Moodle plugin, required other plugin and configure the Moodle plugin to send calculations into the Jobe server

## 1 / 2: Installing Jobe Server

SSH into the Ubuntu 20.04 server where you will install Jobe. For best performance and security **this needs to be** a new server without anything else running on it. Before going further, check that you have Git installed with `git --version`. If it is not found, install Git first:

```
sudo apt update
sudo apt upgrade
sudo apt install git
```

### Installing Docker

Now, we need to install Docker so we can run the Jobe server virtually in the server. Do that using these steps:

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
sudo apt update
apt-cache policy docker-ce
sudo apt install docker-ce
sudo systemctl status docker
```

Now, to be able to use Docker without sudo:

```
sudo usermod -aG docker ${USER}
su - ${USER}
```

Then, just confirm that you are added to the Docker group now with:

```
id -nG
```

### Cloning, building, starting jobeinabox

We will use premade Jobe Docker image called [jobeinabox](https://github.com/Satakunnan-ammattikorkeakoulu/jobeinabox) customized for SAMK to easily install and start the Jobe Docker image.

#### Cloning and building the image

First, we need to clone the image into our computer and move into the cloned folder:

```
cd ~/
git clone https://github.com/Satakunnan-ammattikorkeakoulu/jobeinabox
cd jobeinabox
```

Then, we need to build the image ourself, passing our timezone into it:

```
sudo docker build . -t my/jobeinabox --build-arg TZ="Europe/Helsinki"
```

#### Starting jobeinabox

Then, after we have built the image, we can launch the Docker image in a port 4000:

```
sudo docker run -d -p 4000:80 --name jobe my/jobeinabox
```

It can take even 20 minutes for the image to launch for the first time as it is really large.

#### Viewing Docker image status

You can view the status of the image by executing:

```
sudo docker ps
```

If it still states *Starting*, it is not yet running and ready.

Another way to verify that the Jobe server is started is by using cURL:

```
curl http://127.0.0.1:4000/jobe/index.php/restapi/languages
```

And that's it! Once the Jobe server has been started, we just need to configure the Moodle plugin.

###  Allow Jobe server port 4000 access only into Moodle server

From security perspective it is also great to only allow access to the specific port 4000 from the Moodle server. Do that with:

```
sudo iptables -A INPUT -p tcp --dport 4000 -s MOODLE_SERVER_IP_HERE -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 4000 -j DROP
```

## 2 / 2: Installing CodeRunner Moodle Plugin

1. First, couple Moodle plugins will need to be installed. The plugins are located in Github repository where they can be downloaded as ZIP files into your computer. You can download the plugins as ZIP archives by navigating to links mentioned next and then clicking the green button on topright corner of the page that states "Code" and selecting Download ZIP). Files are located [here](https://github.com/trampgeek/moodle-qbehaviour_adaptive_adapted_for_coderunner) and [here](https://github.com/trampgeek/moodle-qtype_coderunner).
2. Next, login as administrator into the Moodle where you want the CodeRunner installed. From Moodle, navigate to Site Administration -> Plugins tab -> Install Plugins. Install qbehaviour-adaptive-adapter first. Upgrade database when asked.
3. Next, install the actual qtype-coderunner plugin. For configurations, set:

```
Jobe server: JOBE_SERVER_IP_OR_DOMAIN:4000
Jobe API-key: Leave as empty
```

Ready!

## Updating jobeinabox

To update jobeinabox image, follow these steps:

1. Navigate to the location where jobeinabox Github repository have been copied into. By default, this should be: `cd ~/jobeinabox`
2. Run `git pull` to pull latest changes from the Github repository
3. If there were changes, rebuild the Docker image: `sudo docker build . -t my/jobeinabox --build-arg TZ="Europe/Helsinki"`
4. Stop the current Docker image: `sudo docker stop my/jobeinabox`
5. Remove the current Docker image: `sudo docker rmi $(docker images | grep 'my/jobeinabox') -f`
6. Remove the current Ubuntu image spun up for Jobe: `sudo docker rmi $(docker images | grep 'ubuntu') -f`
7. Remove the old containers from the hard drive: `sudo docker container prune`
8. And then deploy the new image: `sudo docker run -d -p 4000:80 --name jobe my/jobeinabox`

## Testing the Installation (as a teacher)

CodeRunner works inside Moodle quizzes. To test the functionality, you need to:

1. Have some test course or create one where you can add a new quiz
2. Add new quiz into that course
3. Add new CodeRunner question into the Test
4. For the CodeRunner question, make these changes (others you can leave for their default values):

```
Question Type: python3
Question name: print "I love programming"
Question text: Print "I love programming"
Answer: print("I love programming")
Expected output (Test case 1): I love programming
```

And click *Save Changes*.

5. Now go back to editing that question you just made (print "I love programming")
6. Scroll down to bottom and click on the *Preview* button.
7. Now in the opened popup, write in the answer field: print("I love programming")
8. It should now validate the answer and on the bottom it should say *Passed all tests!*. If the answer was wrong, it says something like *Your code must pass all tests to earn any marks. Try again.*. If it had an timeout error, error connecting to the Jobe server or some other error, this usually means that the Jobe server is not running or connection to the Jobe server could not be made (Jobe server has port 4000 closed, wrong IP set on CodeRunner Moodle settings or something else)

## Testing the Installation (as a student)

Before we can test the installation as a student, teacher needs to have a course where he/she has created a Quiz that has a single or multiple CodeRunner question(s) added. You can follow the section above (Testing the Installation (as a teacher)) to add an example test with an example question that student can try out.

Testing the installation as a student is simple. First login as a student in Moodle. Now just go to the test Quiz that you had created and where you had at least one CodeRunner question setup. Now complete the question and if it runs OK and you can complete it, it means that the CodeRunner Plugin is working.

# More Tutorials

To find out more tutorials on how to use the CodeRunner Moodle plugin, view the [author's Youtube Channel](https://www.youtube.com/channel/UCDRXp0D9QLBJWxkzjcHTJgA), [official website](https://coderunner.org.nz/), [official documentation](https://coderunner.org.nz/)
