TextScrapper for puzzle#66 btc
A web scraping script designed to search for a specific text within HTML div elements across multiple pages of a website. Utilizes multi-threading for efficient parallel searches and employs a bloom filter to avoid repeated page visits.

Features
Multi-threaded scraping for faster processing.
Uses Bloom Filter to ensure unique page visits.
Customizable User-Agent headers to mimic real browser requests.
Error handling and rate limiting to manage request failures and avoid getting blocked.
Logging to track progress and issues.
Requirements
Python 3.6+
requests library
beautifulsoup4 library
tqdm library
concurrent.futures (part of the Python standard library)
bloom_filter library
logging (part of the Python standard library)
Installation
Install the required libraries using pip:

sh
Copy code
pip install requests beautifulsoup4 tqdm bloom_filter
Usage
Import the necessary modules:

python
Copy code
import requests
from bs4 import BeautifulSoup
from random import randint, choice
from tqdm import tqdm
from concurrent.futures import ThreadPoolExecutor, as_completed
from bloom_filter import BloomFilter
import time
import logging
Initialize the TextScrapper class:

python
Copy code
scrapper = TextScrapper()
Call the find_target_text method with the base URL, target text, and the start page:

python
Copy code
base_url_to_scrape = "https://hashkeys.space/66"
target_text = "13zb1hQbWVsc2S7ZTZnP2G4undNNpdh5so"
start_page = 24349699400214461

result = scrapper.find_target_text(base_url_to_scrape, target_text, start_page, num_parallel_searches=100)
if result:
    logging.info(result)
else:
    logging.info("Target text not found.")
Example
Here's a complete example:

python
Copy code
import requests
from bs4 import BeautifulSoup
from random import randint, choice
from tqdm import tqdm
from concurrent.futures import ThreadPoolExecutor, as_completed
from bloom_filter import BloomFilter
import time
import logging

# Setup logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

class TextScrapper:
    def __init__(self):
        self.bloom_filter = BloomFilter(max_elements=1000000, error_rate=0.001)
        self.user_agents = [
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Firefox/89.0",
            "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.1 Safari/605.1.15"
            # Add more user agents if necessary
        ]

    def from_url(self, url, target_text):
        headers = {
            'User-Agent': choice(self.user_agents)
        }
        try:
            response = requests.get(url, headers=headers)
            if response.status_code == 429:
                time.sleep(randint(1, 5))  # Sleep to handle rate limiting
                return self.from_url(url, target_text)

            response.raise_for_status()
            soup = BeautifulSoup(response.content, 'html.parser')
            divs = soup.find_all('div')
            for div in divs:
                if div.get_text().strip() == target_text:
                    return f"Target text found in div on page {url}"
            return None
        except requests.exceptions.RequestException as e:
            logging.error(f"An error occurred while trying to extract text from the URL {url}: {e}")
            return None

    def find_target_text(self, base_url, target_text, start_page, num_parallel_searches=100):
        error_count = 0
        current_page = start_page

        with ThreadPoolExecutor(max_workers=num_parallel_searches) as executor:
            futures = []

            while current_page <= 36893488147419104:
                if current_page in self.bloom_filter:
                    current_page += 1
                    continue
                
                self.bloom_filter.add(current_page)

                url_to_scrape = f"{base_url}/?page={current_page}"
                logging.info(f"Scraping URL: {url_to_scrape}")
                futures.append(executor.submit(self.from_url, url_to_scrape, target_text))
                
                for future in as_completed(futures):
                    result = future.result()
                    if result:
                        logging.info(result)
                        return result

                    error_count = 0
                
                error_count += 1
                if error_count >= 3:
                    logging.error("Too many errors in a row. Stopping...")
                    break

                futures = [f for f in futures if not f.done()]
                current_page += 1

# Example usage:
base_url_to_scrape = "https://hashkeys.space/66"
target_text = "13zb1hQbWVsc2S7ZTZnP2G4undNNpdh5so"
start_page = 24349699400214461

scrapper = TextScrapper()
result = scrapper.find_target_text(base_url_to_scrape, target_text, start_page, num_parallel_searches=100)
if result:
    logging.info(result)
else:
    logging.info("Target text not found.")
License
This project is licensed under the MIT License - see the LICENSE file for details
