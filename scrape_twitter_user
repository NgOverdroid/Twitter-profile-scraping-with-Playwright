from playwright.async_api import async_playwright
import asyncio
import datetime
import random
import json
import pandas as pd

async def scrape_twitter_user(user_handle):
    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=True)  # Headless mode for Colab
        page = await browser.new_page()
        await page.goto(user_handle)  # Use the provided user_handle

        # Wait for the profile page to load
        await page.wait_for_selector('div[data-testid="UserProfileHeader_Items"]')

        # Scrape the display name
        display_name = await page.query_selector("//div[@data-testid='UserName']/div[1]/div/div[1]/div/div/span/span[1]")
        display_name_text = await display_name.inner_text() if display_name else 'Not Found'

        # Scrape the username
        username = await page.query_selector("//div[@data-testid='UserName']/div[1]/div/div[2]/div/div/div/span")
        username_text = await username.inner_text() if username else 'Not Found'

        # Scrape the followers count
        followers_count = await page.query_selector('a[href*="/verified_followers"] > span > span')
        followers_count_text = await followers_count.inner_text() if followers_count else 'Not Found'

        # Scrape the following count
        following_count = await page.query_selector('a[href*="/following"] > span > span')
        following_count_text = await following_count.inner_text() if following_count else 'Not Found'

        # Địa chỉ
        location_selector = await page.query_selector("//div[@data-testid='UserProfileHeader_Items']/span[1]/span/span")
        location = await location_selector.inner_text() if location_selector else ' '

        await browser.close()

        return {
            "Tên": display_name_text,
            "Username": username_text,
            "Người theo dõi": followers_count_text,
            "Đang theo dõi": following_count_text,
            "Địa chỉ": location
        }

# Example usage
google_sheet_csv_url = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vQ5RuA2SVftGMl3h-iXT-4a5ucgylervibFof0YF4Be_tU9ReJinjMf__hz5hT_W7e2S8Mv9o6OUJF3/pub?output=csv'
df=pd.read_csv(google_sheet_csv_url)
data_list = df.iloc[0:10,0].values.tolist()

with open("info.json", "w") as f:
    json.dump([], f)

n=len(data_list)
def write_json(new_data, filename=None):
    with open(filename,'r+') as file:
        # First we load existing data into a dict.
        file_data = json.load(file)
        # Join new_data with file_data inside emp_details
        file_data.append(new_data)
        # Sets file's current position at offset.
        file.seek(0)
        # convert back to json.
        json.dump(file_data, file, indent = 10)

for i in range(n):
# Directly await the coroutine instead of using asyncio.run()
  user_data = await scrape_twitter_user(data_list[i])
  now = str(datetime.datetime.now())
  write_json({
      "Link" : data_list[i],
      "Tên" : user_data['Tên'],
      "Username" : user_data['Username'],
      "Số lượng following" : user_data['Đang theo dõi'],
      "Số lượng follower" : user_data['Người theo dõi'],
      "Địa chỉ" : user_data['Địa chỉ'],
      "dateupdate" : now
  },filename='info.json')

  with open("info.json") as f:
    data = pd.read_json(f)
  data.to_excel('filename.xlsx',index=False)
