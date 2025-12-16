# Python-Project-Submission
You will create a book suggestion app that integrates various concepts you’ve learned. This project will involve data collection, data processing, user interaction, error handling, and code quality

1. Data Collection
• Web Scraping: Use web scraping techniques to gather book data from a reliable
source. Create a file to store the scraped data. Ensure the data includes book titles,
authors, genres, publication years, popularity, and rankings.
OR
• API Integration: Alternatively, find a suitable API that provides book data. Integrate
this API into your project to fetch the required information.
2. Data Processing:
Pandas: Utilize the Pandas library to read and manipulate the collected data. Ensure the
data is cleaned and structured appropriately for further use.
3. User Interaction:
• Filtered Book List: Implement functionality to filter the book list based on userspecified criteria such as popularity, ranking, or year of publication. Allow the user to
choose the genre and apply filters accordingly.
• Random Book Suggestion: Create a feature that generates a random book
suggestion from the selected genre. Ensure the suggestion is truly random and
includes all relevant details about the book.
4. Error Handling:
Implement robust error handling methods to manage potential issues, such as network
errors, invalid user inputs, and data processing errors. Ensure the app provides informative
error messages and continues to run smoothly.
5. Code Quality:
Write clean, well-documented code. Add comments to explain the purpose of each function
and important code segments. Ensure the code follows best practices for readability and
maintainability.
At the end of this project, you would have created a bot with all the complex concepts you have
learnt throughout the course.



import requests                #import library request for API calls
import pandas as pd            #import library pandas for analysis ans data cleaning
import json                    #import library json for working with JSON (JavaScript Object Notation) data
import ast                     #import library for working with the Abstract Syntax Tree (AST)


def get_books_with_details(query="bestseller", limit=10):
    search_url = "https://openlibrary.org/search.json"                   # Base URL for the Open Library search API
    params = {"q": query, "limit": limit}                          # Parameters sent with the API request:'q' specifies the search term'limit' specifies how many results to return
    
 
    
    try:                                                           # Using a try catch block for network related errors
        search_response = requests.get(search_url, params=params)  #request to the Open Library API with the search parameters
        search_data = search_response.json()                       #Parse the JSON response from the API into a Python dictionary
        docs = search_data.get('docs', [])                         #Extract the list of book entries

        results = []                                                # Initialize an empty list to store the final book details

    except ConnectionError:
        print("⚠️ Network error: Unable to reach Open Library.")   # In case of network error
        
    # Assign rank to the extracted data set
    for rank, doc in enumerate(docs, 1):
        title = doc.get("title")                         # Get book title
        author = doc.get('author_name')                   # Get name of author 
        genre = doc.get("subjects", 'NA')                #Get the Genre of the book
        work_key = doc.get("key")                       # Example: "/works/OL45883W"

        if not work_key:
            continue

       # Fetch work details for subjects / genres
        work_url = f"https://openlibrary.org{work_key}.json"          # Using a work_key extract Subject and genre
        try:
            work_data = requests.get(work_url).json()
            genres = work_data.get("subjects", 'NA')                 # Get genre
            authors= work_data.get("author_name", 'Anonymous')       #Get author name
        except requests.exceptions.RequestException:
            genres = 'NA'
            authors = 'Anonymous'

        # Fetch edition details to get publish year/date
        
        edition_url = f"https://openlibrary.org{work_key}/editions.json?limit=1"
        
        try:
            edition_data = requests.get(edition_url).json()                     #Get the date from books page
            publish_date = None
            
            editions_list = edition_data.get("entries", edition_data.get("docs", []))
            if editions_list:
                
                first_edition = editions_list[0]
                publish_date = first_edition.get("publish_date") or first_edition.get("first_publish_date") #Get publish date
        except requests.exceptions.RequestException:
            publish_date = 'Error fetching date'
            
        

        # Append the result for this single book (rank, title,author,genres,publish_date,Popularity)
        results.append({
            'rank': rank, # This increments correctly with the loop
            "title": title,
            #'author': ', '.join(doc.get('author_name', ['Unknown Author'])),
            'author': author[:1],

            "genres": genres[:1],
            "publish_date": publish_date,
            'Popularity': doc.get('edition_count', 0),     #Get popukarity from the no. of editions of a book
        })

    # The function MUST return AFTER the loop has finished processing all 'docs'
    return results
books = get_books_with_details()
df = pd.DataFrame(books)                #Put convert the table to dataframe using pandas
df                                     #Display the data frame

# =====================================DATA CLEANING==========================================

# Conver the date to datetime datatype
df.publish_date = pd.Series(df.publish_date)
print(df.publish_date)
df.publish_date = pd.to_datetime(df.publish_date, errors='coerce', infer_datetime_format=True)

# Extract the year using the .dt accessor
df.publish_date = df.publish_date.dt.year

#print("Extracted Years (float type):")

df['author'] = df['author'].astype('str')
df['genres'] = df['genres'].astype('str')
df['author'] = df['author'].apply(lambda x: ast.literal_eval(x)[0])
df['genres'] = df['genres'].apply(lambda x: ast.literal_eval(x)[0])

# --- Assume 'df' is your existing DataFrame ---

try:                                                      # We use exception handling to ensure
    
    df['Popularity'] = df['Popularity'].apply(           #Give popularity
        lambda score: 
                      'High' if score >= 400 else (
                      'Moderate' if score >= 200 else (
                      'Low' if score >= 0  else 
                      'INVALID'
                      ))
    )
    print(df[['Popularity']])

except Exception as e:
    
    print("An Error occurred during DataFrame processing:")
    print(f"Exception details: {e}")


df

delimiter = ','   # Use demiliter ',' to select the foremost genre only and ignore the others.

df['genres'] = df['genres'].str.split(delimiter).str[0]
df['publish_date'] = df['publish_date'].astype(str)   #convert publication year to string
df['rank'] = df['rank'].astype(str)                   #convert rank to string
df

#===================================================================================================

def main_menu():                         #Define a menu from where user can select their choices
    
    print("BOOK ANALYSIS MENU")
    print("1.Display all the books in the table")
    print("2. Filter books by genre")
    print("3. Get random book suggestion")
    print("4. Exit")

    choice = input("Enter your choice (1/2/3/4): ").strip()   #User inputs a number which is their choice
    
    switch = {
        '1': lambda: Display_books(df),
        '2': filter_by_genre,
        '3': random_book_suggestion,
        '4': exit_program
    }
    
    action = switch.get(choice, invalid_choice)       #Go to the selection or print invalid selection
    action()

def Display_books(df):                       #Function to display all the books
        print(df)
    
def filter_by_genre():                                            #Function to filter by genre
    choice = input("Do you want to use filters? (y/n): ")
    choice = choice.strip().lower()
    if choice == 'y':
    # Code to run if the user enters 'y'
        genre = input("Enter genre: ").strip()                    #Enter Genre the unser wantes to filter
        filtered_df = df[df['genres'].str.lower() == genre.lower()]   #Accept values irrespective of uppercase or lowercase
        if filtered_df.empty:
            print(f"\nNo books found in the genre '{genre}'.")
        else:
                                                                     # Drop the 'genres' column and save the other info in filtered_df dataframe
            filtered_df = filtered_df.drop(columns=['genres'])

        # Display result
            print(f"\nFiltered Books in '{genre}' genre:\n")
            print(filtered_df)
    
    elif choice == 'n':                                           #if choice n then print thank you
        print("Thank you")

    else:
        print("Invalid input. Please enter 'y' or 'n'.")
          
    #Allow the user to choose the genre and apply filters accordingly
    
    choice1 = input("\n Do you want to perform additional analysis using more parameters? (y/n): ").strip().lower() 

    if choice1 == 'y':
    # Code to run if the user enters 'y'
        print("\nEnter other parameters for further filters")
        unique_popularity = df['Popularity'].str.lower().unique()  # get all unique popularity values in lowercase

        while True:
            popularity = input("Enter Popularity: ").strip().lower()   #user enters popularity
            if popularity in unique_popularity:
                break
            else:
                print(f"Invalid input! Please enter one of: {', '.join(unique_popularity)}")    
        
        
        
        while True:
            publish_date = input("Enter publish Year (1-2026): ").strip()    #publish date taken from the user
            try:
                publish_year = int(publish_date)  # try converting to integer
                if 1 <= publish_year <= 2026:
                    break                         # valid input
                else:
                    print("Invalid year! Please enter a year between 1 and 2026.")
            except ValueError:
                print("Invalid input! Please enter a numeric year.")

        rank = input("Enter rank : ").strip().lower()                 #input rank
    
        #save in filtered_df dataframe
        filtered_df = filtered_df[(filtered_df['Popularity'].str.lower() == unique_popularity) & (filtered_df['publish_date'].str.lower() == publish_date) & (filtered_df['rank'].str.lower() == rank)]

    
        if filtered_df.empty:
            print(
                f"\nNo books found where "
                f"Popularity='{unique_popularity}', "
                f"PublishDate='{publish_date}', "
                f"Rank='{rank}'"
                )
        else:
            print(
                f"\nList of all Books in '{genre}' genres, "
                f"'{unique_popularity}' popularity, "
                f"'{publish_date}' publish year, "
                f"'{rank}' rank:\n"
                )
            print(filtered_df)

    else:
        print("No additional analysis selected \nThank you")

def random_book_suggestion():                                                #Generate a random book suggestion from a genre

    genre = input("Enter genre for random suggestion: ").strip()             #enter genre
    genre_books = df[df['genres'].str.lower() == genre.lower()]
    
    if genre_books.empty:
        print(f"\nNo books found in the genre '{genre}'.")
        return
    
    random_book = genre_books.sample(n=1).iloc[0]                                  #Generate random book
    print(f"🎲 RANDOM BOOK SUGGESTION FROM '{genre.upper()}' GENRE")              #print the details
    
    for column in random_book.index:
        if column != 'genres':
            print(f"{column.replace('_', ' ').title()}: {random_book[column]}")

def exit_program():                                                          # Exit the program 
    print("\nThank you for using the Book Analysis System. Goodbye!")
    exit()

def invalid_choice():                                             # Handle invalid menu choices 

    print("\nInvalid choice. Please enter 1, 2, or 3.")

# Run the main menu
main_menu()

# Optional: Loop to keep the menu running
while True:
    continue_choice = input("\nDo you want to return to the main menu? (y/n): ").strip().lower()
    if continue_choice == 'y':
        main_menu()
    else:
        exit_program()
    

main_menu(df)

