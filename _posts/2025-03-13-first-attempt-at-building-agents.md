---
layout: post
title: >
  First attempt at building AI agents
tags: [ai-agents]
---

# What's an agent?

Hello! It's early 2025 now and everyone has their own definition of an agent. [Here's](https://gist.github.com/simonw/beaa5f90133b30724c5cc1c4008d0654) a compilation of such definitions and here's [Anthropic's opinion](https://www.anthropic.com/engineering/building-effective-agents).

Here's my definition:
An AI agent is a system that has the following characteristics:
- LLM for thinking
- Tools for doing something
- Domain specific data or specialized knowledge 
- Experience (it knows its past mistakes) 
- Ability to tackle new tasks

It autonomously takes actions towards completion of a predefined goal until the goal is accomplished.

# Choosing a problem statement

I have a simple goal: Create an agent that can accomplish a non-software building task. Why? Because I know LLMs are good at coding and the evaluation criteria is right there - on your computer. So, building a coding agent is not much of a proof of concept that agents can excel at useful work.

In the agentic community, a travel agent is a cliche. I thought ofcourse an LLM itself is enough to plan my vacation. But I was proven wrong. Inspite of the internet search capability of GPT, it was hallucinating when asked about flight information.

I saw an opportunity.

# On to building something

Here's the [python file](https://github.com/i-sharma/trip_planner_agent/blob/master/travel_planner.py) if you want to follow along.

I started with the simplest thing there could be - ease my flight search journey while planning a vacation. And surprise, it doesn't require AI. It's just an [API call](https://github.com/i-sharma/trip_planner_agent/blob/fbc6fb5bd0a8536db8a1751d4bf24d1910d4297f/travel_planner.py#L75). 

This method returns a json with cheapest, fastest and best flights from point A to point B on a given date.

Something like this:
~~~json
{
    "CHEAPEST": {
        "departureAirport": "Rajiv Gandhi International Airport",
        "arrivalAirport": "Dehradun Airport",
        "departureTime": "2025-03-28T06:00:00",
        "arrivalTime": "2025-03-28T12:25:00",
        "fare": 9083,
        "legs": [
            {
                "departureAirport": "Rajiv Gandhi International Airport",
                "arrivalAirport": "Delhi International Airport",
                "departureTime": "2025-03-28T06:00:00",
                "arrivalTime": "2025-03-28T08:10:00",
                "flightNumber": "IndiGo 6E 424"
            },
            {
                "departureAirport": "Delhi International Airport",
                "arrivalAirport": "Dehradun Airport",
                "departureTime": "2025-03-28T11:25:00",
                "arrivalTime": "2025-03-28T12:25:00",
                "flightNumber": "IndiGo 6E 2436"
            }
        ]
    },
    "FASTEST": {
        "departureAirport": "Rajiv Gandhi International Airport",
        "arrivalAirport": "Dehradun Airport",
        "departureTime": "2025-03-28T08:50:00",
        "arrivalTime": "2025-03-28T11:15:00",
        "fare": 11502,
        "legs": [
            {
                "departureAirport": "Rajiv Gandhi International Airport",
                "arrivalAirport": "Dehradun Airport",
                "departureTime": "2025-03-28T08:50:00",
                "arrivalTime": "2025-03-28T11:15:00",
                "flightNumber": "IndiGo 6E 422"
            }
        ]
    },
    "BEST": {
        "departureAirport": "Rajiv Gandhi International Airport",
        "arrivalAirport": "Dehradun Airport",
        "departureTime": "2025-03-28T08:50:00",
        "arrivalTime": "2025-03-28T11:15:00",
        "fare": 11502,
        "legs": [
            {
                "departureAirport": "Rajiv Gandhi International Airport",
                "arrivalAirport": "Dehradun Airport",
                "departureTime": "2025-03-28T08:50:00",
                "arrivalTime": "2025-03-28T11:15:00",
                "flightNumber": "IndiGo 6E 422"
            }
        ]
    }
}
~~~


Let's add a layer of complexity to this - the planner will ask the user for a description of their perfect holiday, will suggest top 5 spots and will then suggest the best flight for user. It will then go on to plan a perfect vacation for the user.

The planner is already quite useful, but not an agent yet. The workflows are rigid and coded into the application. To make it an agent, we need to transfer the control over to the LLM.

In the next post, we'll make the planner more powerful.

Here's a sample run of the planner:


~~~
===== Welcome to the Trip Planner! =====

Tell me about your travel preferences (e.g., 'I want to visit someplace quiet during winters in India')

User: I want to go for a monsoon trek in the Himalayas

Finding the perfect destinations for you...

----- Top 5 Recommended Destinations -----

1. Hampta Pass
   Why it's perfect: A stunning trek in Himachal Pradesh, the Hampta Pass offers mesmerizing views of valleys, rivers, and snow-capped peaks, especially during the monsoon season.
   What to do: ['Trekking', 'Photography', 'Camping']

2. Chadar Trek
   Why it's perfect: This unique trek on the frozen Zanskar River offers breathtaking views of the snowy landscape and is a thrilling experience during a spate of rainfall.
   What to do: ['Trekking', 'Ice Climbing', 'Exploring Local Culture']

3. Kedarkantha Trek
   Why it's perfect: Nestled in Uttarakhand, Kedarkantha is famous for its scenic beauty and offers an adventurous trek amidst lush green landscapes during the monsoon season.
   What to do: ['Trekking', 'Camping', 'Wildlife Watching']

4. Triund Trek
   Why it's perfect: A popular trek near Dharamshala, Triund is particularly beautiful during monsoon with lush greenery, fog, and stunning views of the Dhauladhar range.
   What to do: ['Trekking', 'Camping', 'Stargazing']

5. Valley of Flowers
   Why it's perfect: This UNESCO World Heritage site in Uttarakhand boasts an array of colorful flora and mesmerizes hikers with its stunning landscapes rejuvenated by monsoon rains.
   What to do: ['Trekking', 'Nature Walks', 'Flora Exploration']


Select a destination (1-5): 5

Enter your current location to find flights to Valley of Flowers: Hyderabad

Enter your travel date (YYYY-MM-DD): 2025-03-28

Finding flights from Hyderabad to Valley of Flowers on 2025-03-28...
{
    "CHEAPEST": {
        ...
    },
    "FASTEST": {
        ...
    },
    "BEST": {
       ...
    }
}

Generating your personalized trip plan...

===== Your Personalized Trip Plan =====

### 3-Day Trip Plan for Valley of Flowers

#### Travel Dates: March 28 to March 30, 2025

---

### Day 1: Arrival and Exploration of Joshimath

**Morning:**
- **Flight:**
  - Depart from Rajiv Gandhi International Airport at 06:00 via IndiGo 6E 424 (1st Leg).
  - Arrive at Indira Gandhi International Airport, Delhi at 08:50.
  - Change to a connecting flight (2nd leg) departing at around 10:00 and arriving at Dehradun around 12:25.

**Afternoon:**
- **Travel to Joshimath:**
  - Take a taxi from Dehradun Airport to Joshimath (approx. 10-12 hours drive).
  - Lunch can be taken on the way; recommend trying local cuisine such as "Pahadi Aloo" (mountain potatoes) and "Gahat ki Dal" (black gram lentil).

**Evening:**
- **Explore Joshimath:**
  - Visit the Shankaracharya Math and the Narsingh Temple.
  - Stroll around the small market in Joshimath.

**Accommodation:**
- **Hotel:**
  - **GMVN Tourist Rest House** or **The Himalaya Hotel** (mid-range).

---

### Day 2: Trek to Valley of Flowers

**Morning:**
- **Early Breakfast at Hotel:**
  - Enjoy a hearty breakfast featuring local dishes such as "Puri and Chole."
- **Trek to Valley of Flowers:**
  - Depart early for the trek (arrangement of packed lunch may be needed). Travel from Joshimath to Ghangharia, the base for the Valley of Flowers (~14 km).

**Afternoon:**
- **Explore Valley of Flowers:**
  - After reaching Ghangharia, begin the easy trek (about 3-4 km) to the Valley of Flowers. Spend the afternoon discovering the stunning landscapes filled with diverse alpine flora—wildflowers in bloom, picturesque streams, and awe-inspiring mountain vistas.

**Evening:**
- **Return to Ghangharia:**
  - Head back to Ghangharia in the late afternoon and check-in at your accommodation.

**Accommodation:**
- **Hotel:**
  - **Hotel Devlok or Zostel Ghangharia** (budget-friendly hostel-like option).

**Local Cuisine:**
- Dinner at Ghangharia featuring dishes like “Rajma Chawal” (kidney bean curry with rice) and local “BhUT” (corn).

---

### Day 3: Return and Local Sightseeing

**Morning:**
- **Breakfast:**
  - Start your day with breakfast in Ghangharia featuring "Alu Ke Gutke" (spicy fried potatoes).

**Afternoon:**
- **Trek Down:**
  - Trek back to Ghangharia and then onward to Joshimath. Plan to leave early to allow for adequate time for the return trek.

**Lunch:**
- Enjoy lunch at a roadside dhaba (small restaurant) on your way back to Joshimath.

**Evening:**
- **Explore Auli:**
  - If time permits on the way back to Joshimath, take a cable car ride in Auli (also known for its skiing opportunities). Enjoy a picturesque sunset view from the hilltop.

**Return to Joshimath:**
- Dinner at your hotel or nearby restaurant.

**Accommodation:**
- Spend your last night at **GMVN Tourist Rest House** or **The Himalaya Hotel**.

---

### Travel Tips:

1. **Best Time to Visit:**
   - The Valley of Flowers is best visited from late July to early September when wildflowers are in full bloom. Check local weather forecasts and be prepared for rain.

2. **Entry Fees:**
   - Ensure to carry enough cash for entry fees, as ATMs are rare.

3. **Packing:**
   - Carry warm clothing, sturdy trekking shoes, a waterproof jacket, sunscreen, and a reusable water bottle.
   - Bring a good camera or smartphone for the stunning landscapes.

4. **Health Precautions:**
   - Acclimatize well to high altitudes and stay hydrated.
   - Consult your doctor regarding altitude sickness prevention.

5. **Respect Nature:**
   - Follow "Leave No Trace" principles to preserve the delicate ecosystem of the Valley of Flowers.

6. **Permits:**
   - Obtain necessary permits to enter the park (available at the starting point).

7. **Local Guides:**
   - Consider hiring local guides to enhance your experience and support the local economy.

Enjoy your adventure to the mesmerizing Valley of Flowers, where nature’s beauty converges at its finest!

===== Enjoy your trip! =====
~~~