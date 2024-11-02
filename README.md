# Explanation about how scrapping is done:
  The scrapping process is done using Github API in python.
  ## 1.Authorization and Headers: 
    This code sets up the GitHub API token and headers for authentication, bypassing the stricter rate limits on unauthenticated access.
  ### Code:
      GITHUB_TOKEN = 'my_access_token'
      HEADERS = {"Authorization": f"token {GITHUB_TOKEN}"}
  
  ## 2. Helper function to clean company names: 
    'the clean_company_name' function removes the leading "@" sign, removes leading and trailing white space and forces to upper case.
  ### Code:
      def clean_company_name(company):
          if company:
              company = company.strip().lstrip('@').upper()
          return company
     
  ## 3.FETCHING USER DATA: fetch_users
    This function fetches users in a given city (default: Tokyo) with at least a specified follower count from the GitHub API. In fact it fetches the users page by page until there are no more results, pausing for a second after every page to avoid rate limits. For each of the found users, the user's full details such as login, name, location, company, email, etc., are retrieved with an additional API request. The data is kept in a list of dictionaries called users, where the dictionary is for every user.
  ### Code:
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
  
  ## 4.Repository Data Fetching (fetch_repositories):
    This function fetches from the GitHub API all the public repositories of each user, page by page. Details drawn will include name, date created, number of stargazers, and license per repository. The function returns a list of repositories related to the user, and every type of data is stored in dictionaries.
  ### Code:
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
  
  ## 5.Saving Data to CSV:
    save_users_to_csv; save_repositories_to_csv saves user and repository information to two different CSVs. DictWriter is used to ensure each dictionary's keys map to the CSV headers. 
  ### Code:
      Save users to CSV
      def save_users_to_csv(users, filename="users.csv"):
        with open(filename, mode="w", newline="", encoding="utf-8") as file:
          writer = csv.DictWriter(file, fieldnames=users[0].keys())
          writer.writeheader()
          writer.writerows(users)

      Save repositories to CSV
      def save_repositories_to_csv(repositories, filename="repositories.csv"):
          with open(filename, mode="w", newline="", encoding="utf-8") as file:
              writer = csv.DictWriter(file, fieldnames=repositories[0].keys())
              writer.writeheader()
              writer.writerows(repositories)
  
  ## 6.Main Execution (main function): 
    This orchestrates the entire process: fetch the user, save them into a file named users.csv; fetch all repositories from these users, save that to repositories.csv.
  ### Code:
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

# Some Interesting Insights from users.csv data:
  ## 1.Influential users: 
    Some of the users are considered influential or popular based on their large followers and public_repos.
  ## 2.Inactive/Unhireable users:
    Although inactive/unhireable users also make up one-third of total users, so many of its profiles do not look at new chances.
  ## 3.Geo Distribution: 
    The location field describes many users based in major tech hubs, which can offer potential insight into distributions of tech ecosystems.


# Some Interesting Insights from Repository.csv data:
  ## 1.Repositories Popularity: 
    The average number of stars by a stargazer is zero, and the most of the repositories have a minimal count. However, 25% of the repositories have at least 2 stars, and the most popular repository has 58,479 stars.
  ## 2.Most popular programming languages: 
    JavaScript (16.7%), Ruby (10%), Python (7.8%), Go (6.4%), and TypeScript (6.3%). This is a point of showing that developers typically make use of flexible web-centric languages, such as JavaScript and TypeScript.
  ## 3.License Preferences:
    Most of the repositories (54.5%) use MIT License, possibly because of their permissive nature. Among other licenses are Apache 2.0 (15%), and some GNU as well as BSD licenses.


# Recommendation for the Developers:
To further enhance the engagement of developers and to keep up with the trend, introduce a feature known as "Rising Repositories". It not only shows the trending and quality repositories along with the rising star count but also categorizes them under popular programming languages such as JavaScript, Python, and TypeScript. The feature would also have licensing information so that developers are guided in the selection of permissive licenses like MIT or Apache 2.0 to foster collaboration and adoption. This would therefore create an open and encouraging environment of contribution, where the developers who are on the lookout for relevant high quality projects to contribute to, shall contribute on an informed basis.
