# Assignment 2: Functional Design

## Problem Statement
### Problem Domain: Chinese Groceries

I grew up in a Chinese immigrant family, and grocery shopping has always been an important part of our daily life. My parents often struggle to find nearby stores that carry the ingredients they need, especially when moving to a new city. Big platforms like Google Maps or Yelp sometimes help, but it’s not easy to filter for the right kinds of stores or to know which ones reliably carry specific products. For older family members, language barriers and unfamiliarity with local resources make the problem worse. I care about this domain because I’ve seen firsthand how stressful it can be when something as basic as finding groceries becomes a challenge, and I’d like to make that process easier and more reliable.

### Problem: Finding Chinese Groceries

Many Chinese people struggle to find products and produce that they would find in China. Although there are not that many Chinese grocery stores, the few that do exist usually have everything they need. However, it is difficult to locate these stores because they don't appear on map apps. Also, it is hard to search for specific products in Chinese grocery stores online, because most Chinese grocery stores do not support online shopping. 

### Stakeholders

Immigrant shoppers / older adults (primary users): Immigrant shoppers are the most directly affected by the problem, it is difficult to use apps that are designed for English speakers, or apps that are too complex.


Family of shoppers (the ones running the errands): Younger family members often have the burden of assisting immigrant shopper parents with finding stores, the app could reduce that burden on them.
 
 
Chinese grocery store owners (users promoting their store): The app helps promote up-and-coming and smaller international store owners.

### Evidence + Comparables

[Weee!](https://www.sayweee.com/en) Online Asian supermarket app that offers Asian products for delivery. Caters to Chinese, Korean, Japanese, and Southeast Asian audiences.

[Characterising urban immigrants’ interactions with the food retail environment](https://pmc.ncbi.nlm.nih.gov/articles/PMC8565613/) A research article discussing how immigrant communities find and purchase goods differently from the mainstream population because of different preferences and limited English proficiency.

[Food environment interactions after migration: a scoping review on low- and middle-income country immigrants in high-income countries](https://pmc.ncbi.nlm.nih.gov/articles/PMC8825972/) A research paper discussing the barriers low and middle income country immigrants needed to surpass in order to access healthy foods.

[Yami](https://www.yami.com/en) An online retailer catered towards Chinese, Japanese, Korean, and other Asian populations, delivering not only food but also daily products.

[Perceived barriers in accessing food among recent Latin American immigrants in Toronto](https://equityhealthj.biomedcentral.com/articles/10.1186/1475-9276-12-1) A research paper discussing four main difficulties of Latin American immigrant families looking for groceries: limited financial resources; language difficulty; cultural food preferences; and poor knowledge of available community-based food resources and services.


## Application Pitch: ChiHao (吃好)

### Motivation

Chinese immigrant families often struggle to find nearby grocery stores that carry the authentic ingredients they need, and existing tools like Google Maps or Yelp don’t make this process easy or reliable.

### Key Features

1. Smart Store Finder: a search tool that shows nearby Chinese grocery stores on a map, with filters for product type like "rice", "sauces", "produce", store tags like "Halal", "frozen", "Northeastern Chinese", "Southern Chinese". This saves users time and gives them a better idea of where they need to go to find what they want, instead of wasting time going to many stores. Immigrant families and older shoppers can quickly find the right store without trial and error. Store owners gain more visibility for having unique/niche products, and can cater more towards people explicitly looking for them. 


2. Community Reviews and Tips: shoppers can leave reviews or notes on stores' selections and availability, as well as review availability and quality for individual products, something like "freshest bok choy in the area" or "great selection of Lao Gan Ma sauces". This builds collective knowledge for Chinese shoppers in the area, and helps future shoppers have a more streamlined experience. Consumers using the app trust peer experiences and use them to guide purchases, forming a community around Chinese shopping where users can exchange information.


3. Language-friendly Interface: App supports English and Chinese, so users can toggle the interface. Reduces friction for older users and those more comfortable with Chinese. Immigrant families and older users gain independence, no longer relying on younger relatives to translate. Store owners can reach a broader audience.

## Concept Design

### concept StoreDirectory [Store]

**purpose** document and organize stores

**principle** after a store is added to the directory, it can be searched, tagged, and reviewed

**state** 

a set of Stores with:

a storeId String

a name String

an address Location

a set of tags Set\<String\>

a rating Number

a set of reviews Set\<String>

**actions**

addStore(name: String, address: Location): (store: Store)

**requires** no Store with the given name and address exists within the set of Stores

**effect** creates a new Store in the set with empty tags and rating = 0

removeStore(store: Store)

**requires** given Store is in the set of stores

**effect** removes the store from the directory

addTag(store: Store, tag: String)

**requires** given store is in the directory

**effect** adds the given tag to the given Store's set of tags

removeTag(store: Store, tag: String)

**requires** given store is in the directory and has the given tag

**effect** removes the given tag from the store's set of tags

updateRating(store: Store, rating: Number)

**requires** store is in the directory

**effect** replaces the store's current rating with the given rating

addReview(store: Store, text: String, rating: Number)

**effects** adds a review to the given store

getStoreByName(name: String): (store: Store)

**requires** given name is a name of a store in the directory

**effects** returns that Store

getStoreByAddress(address: Location): (store)

**requires** given address is an address of a store in the directory

**effects** returns that Store

getStoresByTags(tags: Set\<String>): (stores: Set\<Store>)

**requires** given tags is are tags of stores in the directory

**effects** returns those Stores

### concept SearchFilter

**purpose** allow users to filter stores by name, tag, and/or location

**principle** after a query is submitted, submits a request to return the store(s) matching the filters

**state**

a query String

a set of activeFilters, Set\<String\>

**actions**

searchByTags(tags: Set\<String>): Request

**effect** returns a request with the given tags

searchByName(name: String): Request

**effect** returns a request with the given address

searchByAddress(address: Location): Request

**effect** returns a request with the given location

### concept Review [User]

**purpose** allow users to report experiences and ratings for stores, and other users to see them

**principle** after a review is added, it can be viewed by others and aggregated into the store's rating

**state** 

a set of Users with:

a set of reviews Set\<String>

a store Store

review text String

a rating Number

**actions** 

createReview(user: User, text: String, rating: Number)

**requires** store exists in the directory

**effect** creates a new review associated with the given user

getReviews(user): (reviews: Set\<String>)

**effect** returns all reviews from the given user

### concept Localization [User]

**purpose** allow users to view the app in their preferred langauge

**principle** after a user selects a language, the interface displays all text/info in that language

**state** 

a set of Users with

a preferredLanguage String

a set of supported languages Set\<String>

**actions**

setLanguage(user: User, language: String)

**requires** given language is in the set of supported languages

**effect** sets the user's preferredLanguage to the given language

getLanguage(user: User): (language: String)

**effect** returns the user's preferredLanguage

### syncs

**addReview**

**when** Review.createReview(user, store, text: String, rating: Number)

**then** Store.addReview(review)

**updateRating**

**when** Review.createReview(user, store, text: String, rating: Number)

Store.addReview(review)

**then** StoreDirectory.updateRating(store, rating: Number)

**searchByName** 

**when**  SearchFilter.searchByName(name)

**then** StoreDirectory.getStoreByName(name)

### Brief Note

The four concepts together provide the essential building blocks for the grocery finder. StoreDirectory is the central source of truth about grocery stores: it records store identities, their names, addresses, tags, ratings, and review references. SearchFilter handles user queries independently; it manages search requests and records the identifiers of matching stores, which are later resolved against the directory via syncs. Review captures user experiences by associating reviews with user IDs and store IDs; its syncs with StoreDirectory keep store ratings and review lists up to date. Localization manages language preferences for each user, ensuring that the interface displays store information and reviews in the language the user has selected.


## UI Sketches

## User Journey

Mei has just moved to Boston. She grew up in a Chinese household and wants to cook familiar meals, but she doesn’t know where to find authentic ingredients like fresh bok choy, soy paste, or Lao Gan Ma sauce. She tried searching “Asian market” on Google Maps, but the results are inconsistent, some stores are far away, while others don’t stock the ingredients she needs. After two frustrating trips, she still hasn’t found a reliable grocery store.

Looking for a better solution, Mei opens the Chinese Grocery Finder app. She uses the search bar and filter buttons: she types “Lao Gan Ma" into the search bar and taps the “Condiments” filter. The app shows three nearby stores, each with an average rating and a short description.

She clicks on a store and sees reviews from other users. One review mentions that the store sells fresh vegetables and another specifically calls out Lao Gan Ma sauce. The store is only a 15-minute walk from her apartment, so she decides to visit.

That afternoon, Mei goes to the store and finds everything she needs. Back at home, she cooks her first proper meal since moving, feeling a sense of comfort and connection. Later, leaves a five star review for that store: “Fresh bok choy, plenty of sauces — very convenient!” Her review contributes to the store’s overall rating and will help the next new student who arrives in the area.
