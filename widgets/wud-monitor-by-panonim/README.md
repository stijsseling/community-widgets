# What's Up Docker Monitor - MENU
The **What's Up Docker Monitor** widget uses the WUD API to display your Docker containers and check if updates are available. It shows the status of each container and allows you to toggle between viewing all containers or only those that need an update. It allows you to choose between forcing wud to refresh container data or fetch existing one. 

---

## Available Versions
I decided to keep old version of wud, because I know that for some people it might look bad or be overcomplicated. I'll try to maintain this version, but I don't promise anything. If you want to request some changes you can find contact info on the bottom. 

### 2.0 - [**Visual Improvment and Overall Info**](wud-main/README.md)
- **Description**: In this version I focused on improving visuals of the widget. I also added overall info wich provides quick container's inspection.
- **Features**:
  - Better look achived by using CSS
  - Added overall info
  - Small bug fixes

    [![2.0 Preview](wud-main/wud_main_preview.png)](wud-main/wud_main_preview.png)

### 1.1 - [**Old Release**](wud-old/README.md)
- **Description**: In v1.1, the widget now uses a `GET` request instead of `POST` to fetch container data, improving performance. 
- **Features**:
  - Simple interface & structure
  - Displays Docker container statuses and update information
  - Improved performance with `GET` request for fetching data
    
    [![1.1 Preview](wud-old/preview_2.png)](wud-old/preview_2.png)
 
---

**Made by:** Artur Flis  
**Contact:** @blue.dev on the project’s Discord
