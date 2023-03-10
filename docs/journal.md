# Journal

Disclaimer: This document is my journal about this project. I am writing all my thoughts trials and errors. This is my way of thinking process. My mind is open-soured in a way. Please note that I am writing this document very fast and there may be too many typos like tis one.


#### 15.12.2022

- Today I have started this project however I had worked and tried to plan a bluprint in last couple of days. Finally the plan seems to be executable and here we are...
- The Story:
    - So last week I went to my family farm located near Salihli (Türkiye). It was the season to harvest the olives from the olive trees. The farm is contracted to another farmer so they are looking after the trees, collecting and seeling the olives by themselves in exchange of a rent fee. The problem with this setup is that there are very limited options for us to audit the contractor if they are looking after the trees and land well enough in the long run. Are they exploiding the trees to get too much yield or they are thinking for a long term try to sustain the trees. As a computer scientist like me it is very hard for me to evaluate their performance in the long run. So I am looking ways to measure proactivelly. During the harvest they were using a vibrating machine. The machine vibrates varius places of the tree and all the olives drops by themselfes so that they can collect with much less effort. There are varius debates that these kind of applications harms trees or it is beneficial for the tree. I do not know! However, it is for sure that we defninetlly need a lot of data about the farm and the progress so the contradictry debates can end and we can move forward in one sustainablle direction for common good. 
    - We have decided not to use the vibration machine but to make sure we need a system to follow the health of the trees individually. Afterall there were thousands (stragelly! this number is also debatable) of trees in the farm which is very hard to track manually. While I was thinking to solve it in a autmated way, I have started to search what I can do with my dji mavic mini drone. The idea is:
        - To shoot videos of every inch of the farm from the top
        - read the video with its geo location information and identified the trees with NN
        - keep them in an organized database with time record so that we can track the status (at least the size for now) of the tree
    - So with the following findings I think the tasks that I have listed above can be done.
- Deepforest -> [here](https://deepforest.readthedocs.io/en/latest/landing.html)  
    - I've started with the second bullet.
    - After couple of searches and fiddle around with [this](https://github.com/A2Amir/Counting-Trees-using-Satellite-Images) network, I've understand that a [Unet](https://en.wikipedia.org/wiki/U-Net) can accomplish task however that repo above is only publishing the network but not the trained model. However thet have a very good struture to train. That seemed promising, but I could not find a dataset to train for that. 
    - I have found [this](https://github.com/nightonion/yosemite-tree-dataset) dataset, however it needs to much slice trials and errors so I put it aside will use it if the other solutions are not working.
    - While I was looking for a dataset I accdiantlly bumped into DeepForest. Which is an open source project and they have a already traing model that can be easily called. They have even made it in a pip package. and a very very good documentation. Thank you... So I've decided to move with this. My discovery is [here](discovery.ipynb)
    - I have tried to predict the accuracy of the model with some samples that have been taken from my farm with my drone. I've discovered that if I some how resize the input image to a rectangle it almost predicts all trees with a reasable amount of miss. So the hardest part is actually been solved.
    - The framework has even an API that outputs the trees as list, and moreover I can append geo location to the list if I provide the images in geotiff format. This will be my identification mechanism for each tree.
- DJI
    - So after solving the hardest problem I started to look how I can extract geo location taged image from the drone.
    - after couple of searches:
        - https://forum.dji.com/thread-140031-1-1.html
        - https://djitelemetryoverlay.com/#products
        - https://djitelemetryoverlay.com/subtitle-extractor/#
        - https://djitelemetryoverlay.com/srt-viewer/
    - this free djitelemetryoverlay tool helped a lot. Thank you sooo much https://github.com/JuanIrache it is great work!...
        - so by exploring these tools I have discovered that actually the video that I shooted with my DJI mavic mini drone, (if you enable the video caption option from the setting before shoothing) records the telemetry data as caption of the video. you can extract with [this](https://djitelemetryoverlay.com/subtitle-extractor/#) tool and analyze the resulting SRT file with [this](https://djitelemetryoverlay.com/srt-viewer/) tool. Look How nice it is:
        ![alt text](srt_overlay_example.gif "My First SRT Overlay experience")
    - So my plan is to some how extract the SRT with my code from the video, and from thoese SRT and images of the video I will create geoTiff files for every second of the video. And I will input those geotiff files to deepforest to get the geo coded list of trees. the last part of the project is to identify the trees using these information and record them in a DB envonrment and all the management/reporting tools that which is another epic. 
    - To create a geotiff with python [this](https://gist.github.com/johannah/f5ff6c09a11c0081383aae755f4f7b7b) might help
        - this code also uses [gdal](https://gdal.org/download.html#development-source) which I need a conda environment to install on my development. So I did that succesfully.
    - So I am looking for ways how to extract SRT from the video by code. I've checked the Juan's repo library but there seems to be no repo for that.
        - watched [this](https://www.youtube.com/watch?v=Lf9ZikB8aJ4) it is already saying that I had discovered by now.
        - I am searching towards [this](https://www.google.com/search?client=safari&rls=en&q=opencv+read+subtitles&ie=UTF-8&oe=UTF-8) direction
            - while looking I've bumped into [this](https://theailearner.com/2018/10/15/extracting-and-saving-video-frames-using-opencv-python/) which I may use to read the video file any way.
        - every solution is about extracting the harcoded subtitle which involves with a lot of OCR buiseness. I do not need that. I have already soft srt data attached to the video. I just simple couple of lines how to read srt which is already there. no luck yet.
        - I am running out of time for today so I will continue later after I am commiting everything.

#### 20.12.2022        

-  extract SRT from the video by code
    - So I've decided to reverse engineer [this](https://djitelemetryoverlay.com/subtitle-extractor/#) 
        - failed to do that! the files wont format properlly.
    - I am looking the API of opencv try to see if there is anything to read the captions.
    - when I am searching how to extract the metadata of a video I've bumper into [this](https://www.thepythoncode.com/article/extract-media-metadata-in-python), however I am getting ffprobe file not found error.
        - I found [this](https://stackoverflow.com/questions/57350259/filenotfounderror-errno-2-no-such-file-or-directory-ffprobe-ffprobe) solution. I need to install the ffmepeg on the computer. python installation may not be enough.
        - Ok I have installed and moved the ff commands into bin of the treecounter environment and it worked
        - however, there is no subtitle information in that result.
        - More interestinglly the resulting probe has attributes called caption and it is 0. as if video has no caption. however I can view the captions in VLC. but not in quicktime.
    - I've mailed Juan, maybe he can help...

#### 21.12.2022

- So I've got response from Juan. Thank you so much for help. [Here](https://superuser.com/questions/583393/how-to-extract-subtitle-from-video-using-ffmpeg) is his forward.
    - Now this is worked, but how can I read it into memory but not into file.
    ```
    ffmpeg -i ~/Downloads/temp_video_for_share.mp4  -map 0:s:0 data/subs.srt
    ```
    - [This](https://stackoverflow.com/questions/69218400/how-to-use-multiple-map-values-in-ffmpeg-python) might help for code version. well could not understand fully how can implement into my project.
    - I need to pipe completelly all the way to the deepforest. instead of input ouput seperatelly.
- Now I have another problem, my discovery notebook stoped working. it locks the kernel while I am executing the deepforest.

#### 28.12.2022

- So I've come to a coworking space in izmir alsancak. Have 2 hours to work..
    - remembering where I've left...
- So I was looking for a way to output ffmpeg result (subtitle) to a variable not and output file. 
    -[this](https://superkogito.github.io/blog/2020/03/19/ffmpeg_pipe.html) might help. In this solution we will not need to install python-ffmpeg module. it will call the commandline directlly. let's try.
        - executing subprocess command seems to be does not work. I think subprocess module did not recognize that ffmpeg is there.
    - How about [this](https://github.com/kkroening/ffmpeg-python/blob/master/examples/README.md#tensorflow-streaming) I've already executed python-ffmpeg let's give it a try.
        - however I am not sure yet how I execute the same command above with python let first accomplish that. this worked. note that ~ tilda symbol in path does not work. you need to give full path.
            ```
            ffmpeg.input("/Users/hakanonal/Downloads/temp_video_for_share.mp4").output("data/subs.srt",map = "0:s:0").run()
            ```
        - no let's pipe it accorindg to the docs...
            - well I could not execute the pipe even with the command line. 
    - Maybe I can directlly convert each seconds of the video as image. and extract the srt file. on my next step I can loop though these to create the geotiff using GAL lib.
        - So extrating srt to file is done. let's extrat each second to image [this](https://superuser.com/questions/135117/how-to-extract-one-frame-of-a-video-every-n-seconds-to-an-image) may help.
            - And it kind of worked. Here is the command line
            ```
            ffmpeg -i /Users/hakanonal/Downloads/temp_video_for_share.mp4 -r 1 output_%05d.png
            ```
    
- So these are the design notes: not a very plesant way of aproach but currentlly this is the best I have managed to make it work.
    - extract subtitles to a file
    - extract each second of video to a image file.
    - loop thorugh subtitles and for each entry
        - create a geotiff file.
        - input each geotiff to deepforest (which is curentlly is not working some reason)
        - execute to db save prosedure ensuring not the record multiple times for the same identified tree.

#### 30.12.2022

- So Last night I've tried [chatGPT](https://openai.com/blog/chatgpt/). "Cheater". Very emotional momnets. So from now on probally most of the code will be based from chatGPT.
    - dialog is in [discovery](discovery.ipynb) notebook.
    - of course it did not work directlly, however it give me a huge jump start. Instead of looking for example codes by searching site by site.
        - I've modified the code according to the errors I have as far as my understanding.

#### 31.12.2022

- I am trying to divde the code into pices and see each steps result and understand.
    - Well I've started to divide, however it did not work initially. I've understand that I need to ask correct questions to chatGPT. 
- Now I need to test if the generated output is readable by DeepForest.
- To re-run the deepforest I am going to destroy the environment and create again.
    - So I am suspecting that when I install some other packages it has clashed versions with others. Some how my freezed requirements may got confused. Hence Since I've solved to create GeoTiff by using rasterio module I did not need GAL. So a clean environment would do great.
    - please next time do not forget: after creating environment move ffmpeg anf ffprobe binary files into environments bin folder.
    - fresh install hepled and I freezed again.
    - Well seems that my generated geo coded tiff recognized by deepforest so task accoplished. Happy new year!
    
#### 02.01.2023

- Well after the acoompliedment of the last year :P, I have a small adjustment. While I am geo coding the image I think I can transform the image size as square. Because my previous try the hit rate of the NN was much hiher when the provided image was square in size. We'll see...
    - thanks to chatGPT and we have modified it quicklly,
- However, the hit rate of the image is pretty bad. out of 13 trees only 4 of them found and also there are 3 false positives.
    - about false positives, I may have to develop some kind of a logic that for the same tree I need to hit the same tree on multiple images so that I would have make sure the hit is not false positive.
- My plan is to consecutivelly predict images and check their geo codes of the same tree on multiple different images.
    - ok I've functioned in the geocode generator.
    - I am repopulating the main program on a fresh notebook [here](../main.ipynb)
- I've almost completed the design of the program. I want to parse subtitles srt file properlly so I can loop though.
    - well with the help of chatGPT, I have started to write a parser and completed a beatiful parser that reads the text subtitle file and outputs all related data within a dataframe.


#### 04.01.2023

- So Is all components ready? Let's see if we can get the voltrun together.
    - 

#### 15.01.2023

- My journal seems to missing. I wonder why? :) Well voltrun is not together I suppose. Let's see what happened. 
    - well We have put a code that calls all the functions that we have planeed and we have ploted the predicted images side by side to check by eye. The performance consistantlly very poor. 
    - I will try to decrease the resolution of the input image to the deepforest and see.
        - Well suprisng turn of events: while what I wanted to decrease the reqoulsition, I used chatGPT again. I had to re-login and my previous questions was removed. So I had to ask the entire question again. When I asked the question I trecommends me a new code that only uses PIL library including the geotransforing. When I also predict the image with deppforest I've %100 percent hit rate. I did that in discovery 
    - I will now implement it to my main and see if it is going to hit on a series of images.
        - So we have definintelly more hits then before. However there are some misses again. I have looked thourgh with eye that if a single tree can be identifed in any of the series. Looks like there is at least one identification. We will see on the next round that if I can identify programaticlly using the geo codes. I am afraid of that the resolution of (refresh rate) goe codes are not good enough. It has to give same coordinate for each different picture for the same tree on different located bounding box and geo location. We'll see how to figure it out.

#### 17.01.2023

- I need to understand the predicted tree's data better so collecting the same data on a single dataframe and save it to csv would be easier for me to analyze? at least I can try to find a pattern in data? let's try
    - ok It turns out that the method that we have created geo images with PIL library is actually not a goe coded image.
- I will try to change resolkution using rasterio.
    - well instead of doing with rasterio I have done it with ffmpeg from the source. the video source png images are directlly being saved as cropped and scaled right away.
- 

#### 18.01.2023

- Wow great day to start coding. chatGPT gave me wonderful cod eexmaple to visualize my polygons on a map. And quicklly iy appears that the polygons that I have generated are wrong. Somehow I need to keep the polygons coordinates are within latitude and longitude specturum.
- So I need to get back to my geocode tiff image generator and or deepforst annonation code.
- Ok as I suspected there just not enugh resolution for to assign a sinlg uniqe latitude and longitude for each tree. To put it another way a single lang-lat combinatoin has may multiple trees in it. So ı need to find a more clver way to identify them.
    - So when I re-think, This would be meaningful question, how many trees are there in a single shot at that same location? I am aware of that there are multiple different shots for the same lat-lon. Taking considering also the height, I may need to consider to approxmizte min-max pixel change and decide if it is another tree or different tree.
    - aditionally, I do not need to annonate the geo code. Even I may not need to geo code the tiff. because I will never have a proper geo coded boundbox polygon on a map for single tree. The resolution is just not enough. However I can define a tree not like polygon but like point.
- So the goal is to derive the lat-long point from the given lat-long of the image and height.
    - Ok we are getting somewhere by tweaking the pixel of the geo coded generator transform object from_origin.
    - Well! I think we properlly mapped the predicted trees on a real map. which is a huge achivement.
    - We will continue, on mapping multiple pictures and continue to explore on this...

#### 19.01.2023

- I will try to put multiple images on the map and see...
    - Well I did that using my main notebook. I've even created seperate features so that I can hide and show individually.
    - It is clear that the origin is same for at least 4 different images so slight changes on the predictions are hepening netween those ~4 images and after that it jumps to another origin. I am stuck at this point.

#### 20.01.2023

- I have started the day thinking about how can get more pricese lat long value, from the given drone meta data. After some chatting with chatGPT using the altidute it seems that we can interpolate (I do not know what it means) and find a more pricise lat long value. Let's discover the results a little bit.
    - So with the help of chatGPT again, with couple of back and forth questions and answers, I have mamanged to create better previse lat and long, however there still same lat and long values for different pictures evn though there is speed on both of the pictures. In short if there is speed the drone should be in different exact position right? chat gpt suggest to use WGS84 ellipsoid model let's see what it is going to suggest?
        - That did not worked either
- Instead I have managed to increase the resolution of the coordinates, by looking at the flight plan and find the direction. By using that direction I have increased the lat or lon with the amount of speed. multiplied by some coefficient. on the map seems it is working. at least the same trees on different pictures seems to be overlapping on the map.