# Ambient Display Project (continued)

This week we continued to work our concept for the ambient display project, pulling data from Strava API and refining an enclosure to compare the user's best and latest 5k times. 

## Electronics
With the wiring and basic coding foundation complete, we moved on to setting up the Strava API. Pulling data from Strava is a little more complicated than pulling from Open-Meteo because it requires 0Auth permissions from the user to access the data. After spending some time with Roopa and ChatGPT in office hours, we were able to gain access to my Strava account (Isabella also did this on her own account at the same time). 

First, I made an application in Strava to get the client id, client secret, and other information:
</br>
</br>
<img width="443" src="https://github.com/user-attachments/assets/333568fe-2346-462a-80c6-b63dc6eddd50" />
</br>
</br>
Then using the local host, I could authorize my own access and get the initial authorization key:
</br>
</br>
<img width="300" src="https://github.com/user-attachments/assets/d96a12b4-c419-42f4-ba3c-4b10d6edc01e" />
</br>
</br>
With this authorization key, I could then use curl requests in the terminal to pull my account-specific data (to test calling the API) and upgrade my access to "read-all" from just "read" (which would allow us to get more detailed data about each run):
</br>
</br>
<img width="600" src="https://github.com/user-attachments/assets/0652c7b5-ad75-48c9-ba3e-f2cba12b5d8e" />
</br>
</br>
So now with all the information we needed to access the Strava API, we could attempt the real code. 
