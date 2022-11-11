[![Ask Me Anything][0a]][0b]
![Open Source Love][0c]

[0a]: https://img.shields.io/badge/Ask%20me-anything-1abc9c.svg
[0b]: https://github.com/open-AIMS/docker-example/issues/new
[0c]: https://badges.frapsoft.com/os/v2/open-source.svg?v=103

**Warning:** For this tutorial, we will provide a Unix-based example, so it should work on any MacOS or Linux terminal.

# Overview

**Goal.** The goal of this tutorial is to make sure that you are able to create a container of your code repository in such a way that yourself in the future, or external collaborators, are able to fully reproduce your work. Most importantly, you will do this in such a way that anyone will be able to reproduce the work without having to worry about differences in software versions today, or 100 years from now.

**Knowledge requirements.** This example is done entirely using free, open-source software. In order for this to work relatively seamlessly, we expect that you have good experience with the R language, git system of version control, and some basic knowledge in bash. If you do not have any of the above skills, this may be a bit out there for you, but please feel free to still give it a go--- we would love to hear your feedback on how to improve the content of this tutorial.

**Major steps.** In this tutorial you will:

1) Pull an existing example git repository hosted on GitHub containing R code that runs a linear model on some dataset, and produces a .pdf output bivariate plot;

2) Learn how to build a Docker container from this repository, and run it locally;

3) Push this container to dockerhub.

# Dependencies

A Unix-based machine, as well as [R](https://cran.r-project.org/), [RStudio](https://posit.co/downloads/) and [Docker][6] installed (see more about these below).

# Approach

First things first.

<h2 id="rpc">
  The reproducibility crisis
</h2>

One of the biggest challenges scientists face today is making sure that their research is fully reproducible. This reproducibility has three main pillars. It starts with a fully transparent plan of the experimental design and hypothesis to be tested ([see more here](https://osf.io/)), moving onto the actual collection of the data ([see more here][5]), to finally making sure that all analyses and output are fully reproducible from scratch using the collected data and computer code. Here we will address the latter form of reproducibility based exclusively on open-source software and free-of-charge on-line platforms.

[5]: https://en.wikipedia.org/wiki/Reproducibility_Project

<h2 id="ops">
  Organised project structure
</h2>

We will assume that you did maintain a record of your original research intention, and that the data is fully collected and, most importantly, **untouched**. Raw data should always be kept as read-only. Literally any modification that is needed to be applied to a dataset can be done via computer code, which helps you keep a fully transparent record of how the data was modified from original version to analysis version used to answer the research question.

Once files are in place, we need to make sure that we maintain everything organised. Here we will follow a simple project directory structure suggested by the [NiceRCode blog][1] (**NB:** this is not the only approach to organising your code). So for the sake of this particular tutorial, we will assume that you have also modularised your code into unit functions ([see examples in the R language][2], but **NB:** modularising your code is a recommendation, not an obligation), and have been maintaining a full record of how the code has been modified through time using [version control with git][3]. Also, make sure you have your own free [GitHub account][4].

[1]: https://nicercode.github.io/blog/2013-04-05-projects/
[2]: https://nicercode.github.io/2014-02-13-UNSW/lessons/10-functions/
[3]: https://nicercode.github.io/2014-02-13-UNSW/lessons/70-version-control/
[4]: https://help.github.com/en/github/getting-started-with-github/signing-up-for-a-new-github-account

## But my open-source coding software keeps changing version!

This is where we start approaching the utility of Docker containers. Many of you may have experienced a situation where the code and project history are fully transparent, have been deposited in an on-line, free repository, but can no longer be reproduced because the software version and associated packages that you originally used are outdated. How can we solve this issue, such that our code will always yield the exact same output, even in a million years from now?

## Enter, Docker container!

As with anything in computer programming, every skill and technique comes packed with horrible lingo. When it comes to Docker containers you will probably see the words image and container being used a lot, so let's go ahead and get those definitions out of our way.

An *image* is a static (unchangeable) file that bundles code and all its dependencies such that your code repository runs reliably on, say, both your original MacBook and your colleague's Windows PC. It contains the necessary system libraries, code, runtime and system tools for this magic to happen. However, an image is just a snapshot which serves as a template to build a *container*. In other words, a container is a running image, and cannot exist without the image, whereas an image can exist without a container.

*Docker* is a containerisation software which allows you to create lightweight, standalone, executable images from which you can create containers to run your code repository and fully reproduce the output. This essentially allows a scientist to isolate their code repository from its environment, solving the third pillar of our [reproducibility crisis](#rpc) section above.

<h2 id="iah">
  I am hooked! How does it work?
</h2>

First of all, you need to download and [install Docker][6] on your machine. This is basically the software that contains all the tricks for you to run your own containers. Once you finished installing it, open the software which will contain OS-specific examples on how to build Docker images.

[6]: https://docs.docker.com/get-docker/

### Downloading some code

1. We will start by forking a public git repository from GitHub. Make sure you are logged into your own GitHub account. Then open the [`open-AIMS/docker-example`][7] repository page on your web browser. At the top right hand corner, there is an option to "Fork" the repository. "Fork" makes a full copy of the `open-AIMS/docker-example` repository on your own account, allowing you to modify the content as much as needed without interfering with the history of the original `open-AIMS/docker-example`.

[7]: https://github.com/open-AIMS/docker-example

2. Now clone the forked repository from GitHub to a local folder of your choice on your machine (below referred to as `path_of_your_choice`). The forked repository will be named `username/docker-example`, where `username` corresponds to your own GitHub account name. Make sure to substitute the appropriate names in the example code below. On your Terminal:

  ```{git clone, engine='bash', results='markdown', eval=FALSE}
  cd path_of_your_choice
  git clone https://github.com/username/docker-example.git
  ```

3. Now locally navigate to the cloned repository

  ```{cd docker, engine='bash', results='markdown', eval=FALSE}
  cd docker-example
  ```

Do not run anything on it just yet. Before we get on to the Docker building part, we need to have a look at the file structure in this repository. This code repository can be run by sourcing `analysis.R`. In brief, this file loads the [ggplot2][8]) package, sources R functions from the `R/functions.R` file, reads some data from the `data` folder, and generates a plot which is saved automatically to a folder named `output`. This code repository is structured following the [NiceRCode guidelines](#ops). You should be able to simply `source("analysis.R")` in R, and inspect the generated output image.

[8]: https://ggplot2.tidyverse.org/

### Building an image

4. We will use the files in this cloned repository to first build an image, and then run a container from this image. In terms of ensuring reproducibility, the key files are `DESCRIPTION`, `Dockerfile`, and `.dockerignore`.

The `DESCRIPTION` contains a general description of what this code repository contains, info about the authors, the license (see [here][9] why you should always include a license with your public repository). It also details the packages dependencies needed to make the code run (in this example, just [ggplot2][8]).

The `Dockerfile` contains a set of instructions that Docker uses to build your image with the correct specifications:

* The three first rows contain information on what version of software you want (we use the [`rocker/verse:3.6.3` image][10] freely provided by the [rocker][11] team), as well as information about yourself (make sure to populate the fields with your own information accordingly).

* The **FROM** command points to rocker, which is in itself an image with all the instructions to install R, RStudio and its system dependencies at a particular version (in this example, 3.6.3).


* The **ARG** command allows for additional user-specified arguments that can be passed to Docker on the command line when building the image.

* In this example, we add the argument `WHEN`, which we will use to specify a precise date from which to install all R package dependencies listed on the `DESCRIPTION`. This is possible by referring to the [MRAN][12] repository.

* The **ENV** / **WORKDIR** / **RUN** commands contain custom-built bash instructions that Docker uses while building the image. All you need to know at this stage is that we are telling Docker to create a folder `/home/rstudio` on which we will save all the files from this code repository.

* This "saving" step is accomplished by the **COPY** command which tells Docker to copy all files/folders from the code repository to the image.

* The `Dockerfile` then tells Docker to install the packages listed within `DESCRIPTION`, and finally runs the **CMD** command which executes tasks when we tell Docker to run a container from the image (see more of this below in step 11).

Please visit [this link][13] for a more in-depth understanding of what the `Dockerfile` is capable of. The `.dockerignore` plays essentially the same role as the `.gitignore` file on your version control system; it lists which files in the repository should **not** be added to the image. See more on why the `.dockerignore` file is important [here][14].

[9]: https://help.github.com/en/github/creating-cloning-and-archiving-repositories/licensing-a-repository
[10]: https://hub.docker.com/r/rocker/verse
[11]: https://github.com/rocker-org/rocker-versioned
[12]: http://mran.revolutionanalytics.com/snapshot/
[13]: https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
[14]: https://docs.docker.com/engine/reference/builder/

5. Now that we're happy about the basic set up, **make sure that Docker is open and running on your local machine.**

  ```{cd docker, engine='bash', results='markdown', eval=FALSE}
  docker --version
  ```

On my machine (as of 2020-10-21), this returns `Docker version 19.03.13, build 4484c46d9d`.

6. We're ready to build the image! Make sure you're on the `docker-example` path, and run on your Terminal (this took 8--10 minutes to run on my iMac):

  ```{docker build, engine='bash', results='markdown', eval=FALSE}
  docker build --build-arg WHEN=2020-03-31 -t docker-example .
  ```

**NB:** Do not forget the trailing `.`. The `--build-args` allows us to pass a value to `WHEN` inside the `Dockerfile`. The `-t` (short for `--tag`) flag allows you to attribute a name to your image (in this case, docker-example). A full list of `build` arguments can be found [here][15].

[15]: https://docs.docker.com/engine/reference/commandline/build/#options

### Running a container

7. Although the image was built, no container has been run or created from this image yet.

  ```{docker search, engine='bash', results='markdown', eval=FALSE}
  # lists all existing images, including docker-example
  docker images -a
  # lists all existing containers
  docker ps -a
  ```

8. If we want to inspect the image and its contents, we need to run it, i.e. essentially firing up a container. We can inspect the container, but for now we won't do anything. Navigate to your machine's home directory first (just to prove the point that the image is now independent of the code repository from which it was built), then run the code below, none line at a time, observing the outputs

  ```{docker run, engine='bash', results='markdown', eval=FALSE}
  cd
  docker run --rm -it --entrypoint=/bin/bash docker-example
  ls -lG
  R
  library(ggplot2)
  packageVersion("ggplot2")
  ```

9. Notice that the R version is 3.6.3 exactly as we specified (3.6.3), and ggplot2 version is 3.3.0, which was the available for R version 3.6.3 on the specified MRAN date. Now quit R and then the container:

  ```{docker quit, engine='bash', results='markdown', eval=FALSE}
  q()
  exit
  ```

10. By default, Docker saves a version of the container to our local machine every time you run the image. This can clutter your disk space, and adding the flag `--rm` ensures that this does not happen, i.e. the container gets deleted after finishing the image run with `exit`. In other words, if you run again

  ```{docker search2, engine='bash', results='markdown', eval=FALSE}
  docker ps -a
  ```
you will see that no containers were saved to your machine. A container would have been saved if it were not for the `--rm` flag. The `-it --entrypoint=/bin/bash` flag allows *interactive* standalone shell access to your container.

11. We can run the container without the interactive mode; Based on the `CMD` line of our `Dockerfile`, Docker will navigate to `/home/rstudio` and run `Rscript analysis.R`

  ```{docker run2, engine='bash', results='markdown', eval=FALSE}
  docker run --rm docker-example
  ```

12. Notice that while it indicates that our plot was produced based on the screen output `Saving 7 x 7 in image`, the output is not made locally available to us, and the container was deleted given the `--rm` flag. You can inspect a new container to check that the original image remains unaltered (i.e. no `output` folder exists).

  ```{docker run3, engine='bash', results='markdown', eval=FALSE}
  docker run --rm -it --entrypoint=/bin/bash docker-example
  ls -lG
  exit
  ```

13. So, although the previous step was necessary for learning, it was of no use to us in practical terms. The practical solution is to run a new container associated with a local volume, so the output gets saved locally. To do that, we need to create a local directory, e.g. `outputdocker` which will serve as the volume onto which the `output` folder inside the running container gets attached. The volume is indicated with the `-v` flag followed by `local_volume:container_directory`.

  ```{docker run4, engine='bash', results='markdown', eval=FALSE}
  mkdir outputdocker
  docker run --rm -v ~/outputdocker:/home/rstudio/output docker-example
  ```

**NB:** With the above code, Docker creates `/home/rstudio/output` automatically, so when Docker runs `analysis.R`, R will return a warning message stating that the folder `output` already exists because `analysis.R` also tries to create a folder `output`; just ignore it.

14. After the above, you should see the `myplot.pdf` also saved to your local folder `outputdocker`. Alternatively, everything can be run interactively attached to the local `outputdocker` volume at once, i.e. combining the above steps. open your local files explorer, remove the output file `myplot.pdf` and notice the changes as you run this code --- run it, but don't quit interactive mode just yet

  ```{docker run5, engine='bash', results='markdown', eval=FALSE}
  docker run --rm -it --entrypoint=/bin/bash -v ~/outputdocker:/home/rstudio/output docker-example
  Rscript analysis.R
  ```

15. You should now see the `myplot.pdf` back in `outputdocker`. You can keep exploring and running anything you want on the container, including producing more code-produced files to `output`, and, by extension, `outputdocker`. For instance, remove the `myplot.pdf` from the container

  ```{docker rm, engine='bash', results='markdown', eval=FALSE}
  rm output/myplot.pdf
  ```

In doing so, it also gets removed from `outputdocker`. The other way around also works; if you delete `myplot.pdf` from `outputdocker` on your machine, it will also be deleted from `/home/rstudio/output` in the container.

**NB:** while the container is running you won't be able to delete the `output` folder because it is linked to `outputdocker`. Don't forget to exit the container

  ```{docker exit, engine='bash', results='markdown', eval=FALSE}
  exit
  ```

You can also have a more advanced customised `Dockerfile` to, for example, run a container from an RStudio session via your web browser (see this [great][16] example by Drs [Daniel Falster][17] and [Saras Windecker][18].

[16]: https://github.com/dfalster/Westoby_2012_JTB_sapwood_model
[17]: https://danielfalster.com/
[18]: https://www.smwindecker.com/

## This is awesome! How can I share my container with colleagues?

This is essentially the last part of our tutorial. You may be happy with building your Docker container locally, but you may also want to make it accessible to your colleagues who are less versed in these tools. You have two main options, in increasing level (not that much) of time investment:

A) One option (harder for colleagues, easier for you) would be for them to also follow steps 2--13 above (they don't need to fork your GitHub repo as long as they don't try to push back to it -- they won't have the permissions to do so unless you give it to them).

B) Another option is to push the container to [dockerhub][19], similar to how one would push code to their repository on GitHub. To do so, first make sure you have created an account with dockerhub, and that you are logged into this account locally on their Docker app. To make things consistent and easy to remember, I would try to have the account name be the same as your GitHub account name. Then, simply go back to the Terminal and type:

[19]: https://hub.docker.com/

  ```{docker push, engine='bash', results='markdown', eval=FALSE}
  docker tag docker-example username/docker-example
  docker push username/docker-example
  ```

remember to replace `username` with your actual dockerhub account name. You can check that the container is now hosted on your [dockerhub repositories][20]. Your colleagues can then pull the Docker container locally on their machines, and can simply run it (i.e. step 13 in the [above section](#iah)). That requires them to also have the Docker app installed on their machines. They have to pull the Docker container you created and run it:

[20]: https://hub.docker.com/repositories

  ```{docker clonerun, engine='bash', results='markdown', eval=FALSE}
  cd
  docker pull username/docker-example
  mkdir outputdocker
  docker run --rm -v ~/outputdocker:/home/rstudio/output username/docker-example
  ```

**NB:** By default, the commands above will push a public repository to dockerhub. dockerhub will only provide the user with one private repository. You need to pay a fee to have access to multiple private repositories if needed to share containers in private.

# Limitations

Hard to tell apart from steep learning curve. One needs to know some programming language (e.g., R, python), Unix-based commands in bash, then Docker language as well.

## Acknowledgements

We would like to thank Drs [Daniel Falster][17] and [Saras Windecker][18] for providing us with a [great example][16] on how to implement this reproducibility framework. Also, we thank the [rocker][11] team for making this much possible.
