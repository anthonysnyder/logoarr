# Logoarr
🎬 **Logoarr**: (Partially) Automate your movie and TV show logo collection! Logoarr is a Flask app that scrapes directories and downloads high-quality transparent logos for your **movies** and **TV shows** from TheMovieDB (TMDb). Designed for seamless integration with Docker or Synology NAS. Optional Slack integration for instant notifications!

Welcome to **Logoarr**, a Flask-based application designed to keep your movie and TV show libraries visually appealing by automatically downloading high-quality transparent logos. Whether managing a vast collection on your Synology NAS or organizing a smaller local library, Logoarr is here to help. With support for movies and TV shows, you can streamline logo management across all your media. Additionally, you can opt-in for Slack notifications to get instant updates when new logos are added!

## Features

	•	Automated Logo Downloading: Automatically fetch transparent logos for movies and TV shows from TMDb based on your directory structure.
	•	Multi-Directory Support: Seamlessly manage multiple directories for both movies and TV shows.
	•	Slack Integration (Optional): Get notified instantly in your Slack channel when a logo is downloaded.
	•	Docker Compatible: Built with Docker in mind, especially for Synology NAS setups.
	•	Media Collection Dashboard: View all movies and TV shows in your collection with their corresponding logos.
	•	Search Functionality: Search for movies and TV shows directly from TMDb based on user input.
	•	Logo Update: Select and update missing logos for your movies or TV shows.
	•	Smooth Scrolling with Anchors: Automatically scroll to the selected media in the list after a logo is updated.
	•	PNG Format with Transparency: All logos are saved in PNG format to preserve transparency for overlay effects.

## Prerequisites

	•	Docker installed on your system.
	•	A Synology NAS or any system capable of running Docker containers.
	•	A valid TMDb API key.
 	•	If you have existing logo.png in your movie folders, thumbnails will be automatically generated as logo-thumb.png for faster loading.
	•	(Optional) Slack webhook URL for notifications.
 **IMPORTANT NOTE**: This app automatically creates logo-thumb.png thumbnails when downloading logos to reduce load times on large libraries. 
 
 ## Getting Started

## 1. Clone the Repository:
```
git clone https://github.com/anthonysnyder/Logoarr.git
cd Logoarr
```
## 2. Setup Docker Compose:
Create a docker-compose.yml file in the root directory of the project:
Note: Make sure to replace placeholders with your actual data/paths to files.

```
services:
  logoarr:
    image: swguru2004/logoarr:latest
    container_name: logoarr
    ports:
      - "1453:5000"
    environment:
      - TMDB_API_KEY=your_tmdb_api_key_here
      - PUID=your_puid_here
      - PGID=your_pgid_here
      - SLACK_WEBHOOK_URL=your_slack_webhook_url_here
    volumes:
      - /path/to/movies:/movies
      - /path/to/kids-movies:/kids-movies
      - /path/to/tv:/tv
      - /path/to/kids-tv:/kids-tv
    networks:
      - bridge_network

networks:
  bridge_network:
    driver: bridge
```
## 3.	Start the Application:
```
docker-compose up -d
```
##	4.	Access the Application:
Open your browser and go to http://localhost:5000 or your NAS IP to start using Logoarr.

## Slack Integration (Optional)

To enable Slack notifications, add your Slack Webhook URL in the SLACK_WEBHOOK_URL environment variable in the docker-compose.yml file.
```
environment:
  - SLACK_WEBHOOK_URL=your_slack_webhook_url_here
```
## Screenshots

Note: Screenshots will be updated to reflect Logoarr functionality.

## Contributing

Feel free to fork the repo and submit pull requests. Your contributions are welcome!

## License

This project is licensed under the MIT License. See the LICENSE file for more details.

## Credits

Based on the original [Postarr](https://github.com/anthonysnyder/Postarr/) project, modified to download logos instead of posters.
