# <ins> Github users in Tokyo with over 200 followers </ins>
## <ins> Files : </ins>
  1.  [users.csv](https://github.com/Abhishek-IITM2026/TDS-Project-1/blob/main/users.csv) : Contains information about the GitHub users in Tokyo with over 200 followers. It consists of 541 users.
  2.  [repositories.csv](https://github.com/Abhishek-IITM2026/TDS-Project-1/blob/main/repositories.csv) : Contains data of 66,427 public repositories of the users in [users.csv](https://github.com/Abhishek-IITM2026/TDS-Project-1/blob/main/users.csv).
  3.  [gitscrapper.py](https://github.com/Abhishek-IITM2026/TDS-Project-1/blob/main/gitscrapper.py) : Python script used to collect this data
  4.  [Analysis.ipynb](https://github.com/Abhishek-IITM2026/TDS-Project-1/blob/main/Analysis.ipynb) : This is the colab file that stores the codes used to analyze both csv files and answer the questions.
  5.  [README.md](https://github.com/Abhishek-IITM2026/TDS-Project-1/blob/main/README.md): This file contains a summary of this project, findings and recommendations.

![](https://github.com/Abhishek-IITM2026/TDS-Project-1/blob/main/data_analytics.gif)

## <ins> Explanation about how scrapping is done : </ins>
  The scrapping process is done using Github API's end point "https://api.github.com/search/users?q=location:Tokyo+followers:>200&page={page}&per_page=100" to get a list of all users that are from Tokyo and have more than 200 followers. Then "https://api.github.com/user/{user_id}" is used to get all the required details of the users.
  For repositories, "https://api.github.com/users/{user_login}/repos?per_page=100&page={page}" is used to get details about each repository present for every user, storing all the data in a csv file using python.
  ### The explaination for the python code is given below:
  ###  1.Authorization and Headers : 
  This code sets up the GitHub API token and headers for authentication, bypassing the stricter rate limits on unauthenticated access.
  #### Code :
      GITHUB_TOKEN = 'my_access_token'
      HEADERS = {"Authorization": f"token {GITHUB_TOKEN}"}
  
  ### 2.  Helper function to clean company names : 
  'the clean_company_name' function removes the leading "@" sign, removes leading and trailing white space and forces to upper case.
  #### Code :
      def clean_company_name(company):
          if company:
              company = company.strip().lstrip('@').upper()
          return company
     
  ### 3.FETCHING USER DATA :
  'fetch_users' function fetches users in a given city (default: Tokyo) with at least a specified follower count from the GitHub API. In fact it fetches the users page by page until there are no more results, pausing for a second after every page to avoid rate limits. For each of the found users, the user's full details such as login, name, location, company, email, etc., are retrieved with an additional API request. The data is kept in a list of dictionaries called users, where the dictionary is for every user.
  #### Code :
      def fetch_users(city="Tokyo", min_followers=200):
          users = []
          page = 1
          while True:
              url = f"https://api.github.com/search/users?q=location:{city}+followers:>{min_followers}&page={page}&per_page=100"
              response = requests.get(url, headers=HEADERS)
              data = response.json()
                      for user in data['items']:
                  user_url = user['url']
                  user_response = requests.get(user_url, headers=HEADERS)
                  user_data = user_response.json()
                  users.append({
                      'login': user_data['login'],
                      'name': user_data['name'],
                      'company': clean_company_name(user_data['company']),
                      'location': user_data['location'],
                      'email': user_data['email'],
                      'hireable': user_data['hireable'],
                      'bio': user_data['bio'],
                      'public_repos': user_data['public_repos'],
                      'followers': user_data['followers'],
                      'following': user_data['following'],
                      'created_at': user_data['created_at'],
                  })
              page += 1
              time.sleep(1)
  
  ### 4.Repository Data Fetching :
  'fetch_repositories' function fetches from the GitHub API all the public repositories of each user, page by page. Details drawn will include name, date created, number of stargazers, and license per repository. The function returns a list of repositories related to the user, and every type of data is stored in dictionaries.
  #### Code :
      def fetch_repositories(user_login):
          repositories = []
          page = 1
          while True:
              url = f"https://api.github.com/users/{user_login}/repos?per_page=100&page={page}"
              response = requests.get(url, headers=HEADERS)
              repo_data = response.json()
                      for repo in repo_data:
                  repositories.append({
                      'login': user_login,
                      'full_name': repo['full_name'],
                      'created_at': repo['created_at'],
                      'stargazers_count': repo['stargazers_count'],
                      'watchers_count': repo['watchers_count'],
                      'language': repo
  
  ### 5.Saving Data to CSV :
  'save_users_to_csv' and 'save_repositories_to_csv' saves user and repository information to two different CSVs. DictWriter is used to ensure each dictionary's keys map to the CSV headers. 
  #### Code :
    
      def save_users_to_csv(users, filename="users.csv"):
        with open(filename, mode="w", newline="", encoding="utf-8") as file:
          writer = csv.DictWriter(file, fieldnames=users[0].keys())
          writer.writeheader()
          writer.writerows(users)

      def save_repositories_to_csv(repositories, filename="repositories.csv"):
          with open(filename, mode="w", newline="", encoding="utf-8") as file:
              writer = csv.DictWriter(file, fieldnames=repositories[0].keys())
              writer.writeheader()
              writer.writerows(repositories)
  
  ### 6.Main Execution (main function) : 
  This orchestrates the entire process: fetch the user, save them into a file named users.csv; fetch all repositories from these users, save that to repositories.csv.
  #### Code :
      def main():
        print("Fetching users...")
        users = fetch_users()
        save_users_to_csv(users)
        print(f"Saved {len(users)} users to users.csv")
    
        print("Fetching repositories...")
        all_repositories = []
        for user in users:
            user_repos = fetch_repositories(user["login"])
            all_repositories.extend(user_repos)
            print(f"Fetched {len(user_repos)} repositories for user {user['login']}")
    
        save_repositories_to_csv(all_repositories)
        print(f"Saved {len(all_repositories)} repositories to repositories.csv")

      if __name__ == "__main__":
          main()

## <ins> Some Interesting Insights from users.csv data : </ins>
  ### 1.Influential users : 
  Some of the users are considered influential or popular based on their large followers and public_repos.
  ### 2.Inactive/Unhireable users :
  Although inactive/unhireable users also make up one-third of total users, so many of its profiles do not look at new chances.
  ### 3.Geo Distribution : 
  The location field describes many users based in major tech hubs, which can offer potential insight into distributions of tech ecosystems.
  ### 4.Users Bio:
  One of the most startling findings of this analysis was that the developers with very short and precise bios received more followers than those developers with very long and elaborate descriptions. Short descriptions have obviously made the profiles more easily accessible and appealing, not to readers, but those who rapidly scan through their profiles.

## <ins> Some Interesting Insights from repository.csv data : </ins>
  ### 1.Repositories Popularity : 
  The average number of stars by a stargazer is zero, and the most of the repositories have a minimal count. However, 25% of the repositories have at least 2 stars, and the most popular repository has 58,479 stars.
  ### 2.Most popular programming languages : 
  JavaScript (16.7%), Ruby (10%), Python (7.8%), Go (6.4%), and TypeScript (6.3%). This is a point of showing that developers typically make use of flexible web-centric languages, such as JavaScript and TypeScript.
  ### 3.License Preferences :
  Most of the repositories (54.5%) use MIT License, possibly because of their permissive nature. Among other licenses are Apache 2.0 (15%), and some GNU as well as BSD licenses.


## <ins> Recommendation for the Developers : </ins>
  ### 1. Capitalize on Popular Languages:
  Popular languages that developers may consider contributing to include JavaScript, Ruby, Python, Go, and TypeScript, as these are the ones dominating the rankings. Developers can also be part of projects in popular languages because more people show interest in those, hence increasing their profile visibility along with community involvement.
  ### 2. Participating Actively :
  onsider dedicating time on weekends to engage actively on GitHub by contributing to personal projects or open-source initiatives. Weekend activity allows you to focus on enhancing your own repositories, showcase skills, and build a visible track record of contributions. By regularly pushing updates, fixing bugs, or adding features, you make your profile more attractive to potential collaborators and employers. Additionally, weekends provide a chance to engage in community discussions, review others’ work, and explore trending repositories to stay current with new technologies. This consistent activity can enhance your profile’s appeal, expand your network, and boost your standing within the GitHub community.
  ### 3. Showcase Successful Projects to Attract Contributors :
  Popular repositories might also be highlighted to drive attention to the best practices and attract potential contributors. Success stories might be shared on social platforms or posted on developer forums for more enhancement of community involvement and inspiring new contributions.
  ### 4. Keep your bio short, focused, and impactful:
  Considering that shorter word bios generate more followers, try to give a hint of key skills, interests, or the things you are currently working on. Use words like key programming languages, specialties, ("Machine Learning Enthusiast"), or notable connections to attract a reader without being too elaborate. It can really be a plus to engagement and followers, given how most users form opinions over profile previews on this website.
