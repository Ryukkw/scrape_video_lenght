import re
import time
import pandas as pd
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By

# Path to your ChromeDriver executable
CHROME_DRIVER_PATH = "C:/chromedriver/chromedriver.exe"  # Adjust this path

def convert_duration_to_seconds(duration_str):
    """Convert a duration string (MM:SS) to total seconds."""
    minutes, seconds = map(int, duration_str.split(':'))
    return minutes * 60 + seconds

def format_seconds_to_hms(total_seconds):
    """Convert total seconds to HH:MM:SS format."""
    hours = total_seconds // 3600
    minutes = (total_seconds % 3600) // 60
    seconds = total_seconds % 60
    return f"{hours:02}:{minutes:02}:{seconds:02}"

def scrape_and_sum_video_durations_with_selenium(page_url):
    """Scrape video durations from the VK page using Selenium."""
    # Initialize the Chrome WebDriver
    service = Service(CHROME_DRIVER_PATH)
    driver = webdriver.Chrome(service=service)

    # Open the VK page
    driver.get(page_url)
    
    # Let the page load initially
    time.sleep(5)  # Adjust this time if needed
    
    # Scroll until no more new content is loaded
    last_height = driver.execute_script("return document.body.scrollHeight")
    
    while True:
        # Scroll down to the bottom of the page
        driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        
        # Wait for new content to load
        time.sleep(5)
        
        # Calculate new scroll height and compare it with the last scroll height
        new_height = driver.execute_script("return document.body.scrollHeight")
        if new_height == last_height:
            break  # No more new content
        last_height = new_height
    
    # Now that the page is fully loaded, find all video duration elements
    duration_elements = driver.find_elements(By.XPATH, '//span[@data-testid="video_card_duration"]')
    
    # Total seconds accumulator
    total_seconds = 0
    
    # Regular expression to match valid MM:SS duration format
    duration_pattern = re.compile(r'^\d{1,2}:\d{2}$')

    # Process all durations
    video_count = 0
    for element in duration_elements:
        duration_str = element.text.strip()
        print(f"Found duration: {duration_str}")
        
        # Only process strings that match the MM:SS format
        if duration_pattern.match(duration_str):
            total_seconds += convert_duration_to_seconds(duration_str)
            video_count += 1
        else:
            print(f"Skipping invalid duration format: {duration_str}")
    
    # Format the total seconds into HH:MM:SS
    total_duration = format_seconds_to_hms(total_seconds)
    
    # Close the browser
    driver.quit()

    # Return the total duration and video count
    return total_duration, video_count

def process_groups_and_export_to_excel(group_urls):
    """Process each VK group, get video durations, and export results to Excel."""
    results = []

    # Loop through each VK group
    for group_url in group_urls:
        # Convert the group URL to the video URL
        video_page_url = f'https://vk.com/video/@{group_url.split("/")[-1]}/all'
        print(f"Processing group: {group_url} | Video URL: {video_page_url}")
        
        # Scrape the videos from the video page
        total_duration, video_count = scrape_and_sum_video_durations_with_selenium(video_page_url)
        print(f"Group: {group_url} | Videos: {video_count} | Total Duration: {total_duration}")
        
        results.append({
            'Group': group_url,
            'Number of Videos': video_count,
            'Total Duration': total_duration
        })

    # Convert results to a DataFrame
    df = pd.DataFrame(results)

    # Save DataFrame to an Excel file
    output_file = 'vk_video_data.xlsx'
    df.to_excel(output_file, index=False)

    print(f"Data exported successfully to {output_file}")

# Example list of VK groups (without the /video/@ part)
group_urls = [
    'https://vk.com/graces_world',  # Replace with actual group URLs
    'https://vk.com/club221764047'
]

# Process the groups and export the data to an Excel file
process_groups_and_export_to_excel(group_urls)
