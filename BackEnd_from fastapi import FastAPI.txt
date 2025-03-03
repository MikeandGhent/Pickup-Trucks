from fastapi import FastAPI
import requests
from bs4 import BeautifulSoup

app = FastAPI()

def scrape_car_prices():
    url = "https://www.examplecarsite.com/cars-for-sale"  # Replace with actual site
    headers = {"User-Agent": "Mozilla/5.0"}  # Mimics a real browser request
    
    response = requests.get(url, headers=headers)
    if response.status_code != 200:
        return {"error": "Failed to retrieve page"}
    
    soup = BeautifulSoup(response.text, 'html.parser')
    car_listings = []
    
    for listing in soup.select(".car-listing"):  # Adjust selector based on website's HTML
        car_name = listing.select_one(".car-title").text.strip()
        price = listing.select_one(".price").text.strip()
        mileage = listing.select_one(".mileage").text.strip() if listing.select_one(".mileage") else "Unknown"
        year = listing.select_one(".year").text.strip() if listing.select_one(".year") else "Unknown"
        location = listing.select_one(".location").text.strip() if listing.select_one(".location") else "N/A"
        contact = listing.select_one(".contact a")["href"] if listing.select_one(".contact a") else "#"
        
        if "pickup" in car_name.lower():  # Only include pickup trucks
            car_listings.append({
                "name": car_name,
                "price": price,
                "mileage": mileage,
                "year": year,
                "location": location,
                "contact": contact
            })
    
    return car_listings

@app.get("/cars")
def get_car_prices():
    return scrape_car_prices()

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
