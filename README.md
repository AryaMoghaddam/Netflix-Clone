<!--- STARTEXCLUDE --->
# 🎓 Netflix Clone using AstraDB and GraphQL

[![Gitpod ready-to-code](https://img.shields.io/badge/Gitpod-ready--to--code-blue?logo=gitpod)](https://gitpod.io/from-referrer/)
[![License Apache2](https://img.shields.io/hexpm/l/plug.svg)](http://www.apache.org/licenses/LICENSE-2.0)
[![Discord](https://img.shields.io/discord/685554030159593522)](https://discord.com/widget?id=685554030159593522&theme=dark)

*50 minutes, Intermediate, [Start Building](#1-login-or-register-to-astradb-and-create-database)*

A simple **ReactJS** Netflix homepage clone running on *AstraDB* that leverages a GraphQL API with *paging* and *infinite scrolling.* The materials has been built with the collaboration of [Ania Kubow](https://www.youtube.com/channel/UC5DNytAJ6_FISueUfzZCVsw) and Datastax developer advocates team.

<!--- ENDEXCLUDE --->
- [Live Demo](https://workshop-graphql-netflix.netlify.app/) or [Video Walkthrough](https://imgur.com/3ns3UJB)

![image](./img/ui.png)

## 🎯 Objectives
* Deploy a Netflix clone to production
* Learn **GraphQL API** and how to use it with a database to create the tables and navigate the data.
* Learn about **paging** and **infinite scrolling** in web ui
* Leverage Netlify and DataStax AstraDB

## Materials 

- [Slide deck](./slides/slides.pdf)
- [Discord chat](https://bit.ly/cassandra-workshop)
- [Questions and Answers](https://community.datastax.com/)


## Table of contents

### Part I - DB Setup & Data Ingest
1. [Create AstraDB Instance](#1-login-or-register-to-astradb-and-create-database)
2. [Create a security token](#2-create-a-security-token)
3. [Create table **genre** with GraphQL](#3-create-table-genre-with-graphql)
4. [Insert data in **genre**  with GraphQL](#4-insert-data-in-the-table-with-graphql)
5. [Retrieve values of **genre** table](#5-retrieving-list-of-values)
6. [Create **movie** table](#6-creating-a-movies-table)
7. [Insert values in **movie** table](#7-insert-values-in-movie-table)
8. [Retrieve values from **movie** table](#8-retrieve-values-from-movie-tables)
9. [Load a CSV DataSet](#9-load-a-csv-dataset)

###  Part II - Deploy to Production
1. [Deploy to Netlify](#1-deploy-to-netlify)
2. [Clone your GitHub repository](#2-clone-your-github-repository)
3. [Launch GitPod](#3-launch-gitpod-ide)
4. [Install the Netlify CLI](#4-install-the-netlify-cli-command-line-interface)
5. [Retrieve application token to securely connect to the database](#5-generate-application-token-to-securely-connect-to-the-database)
6. [Configure Environment Variables and Install Dependencies](#6-configure-and-connect-database)
7. [Launch your app](#7-launch-your-app)
8. [Connect Netlify to your site](#8-connect-netlify-to-your-site)
9. [Deploy to production](#9-deploy-to-production)

### Extra resources
[What is JamStack?](jamstack.md)

[Video tutorial with Ania Kubow](#video-tutorial-with-ania-kubow)

# Part 1 - DB Setup & Data Ingest

## 1. Login or Register to AstraDB and create database

> *When creating your instance use the promotion code **ANIA200** to get 200$ of free credit allowing you about 30 million writes + 30 Million reads  + 50GB a month of monthly storage!!*

**`ASTRADB`** is the simplest way to run Cassandra with zero operations at all - just push the button and get your cluster. No credit card required, $25.00 USD credit every month, roughly 5M writes, 30M reads, 40GB storage monthly - sufficient to run small production workloads.  

✅ **Step 1a:** Click the button to login or register with Datastax. You can use your `Github`, `Google` accounts or register with an `email`.

_Make sure to chose a password with minimum 8 characters, containing upper and lowercase letters, at least one number and special character_

<a href="http://dtsx.io/netflix-workshop-1"><img src="img/create_astra_db.png?raw=true" /></a>
- <details><summary>Show me!</summary>
    <img src="https://github.com/datastaxdevs/workshop-spring-stargate/raw/main/images/tutorials/astra-create-db.gif?raw=true" />
</details>

**creating the database**
|Field| Value|
|---|---|
|**database name**| `netflix_workshop_db` |
|**keyspace**| `netflix_keyspace` |
|**Cloud Provider**| *Use the one you like, click a cloud provider logo,  pick an Area in the list and finally pick a region.* |

_You can technically use whatever you want and update the code to reflect the keyspace. This is really to get you on a happy path for the first run._

You will see your new database `pending` in the Dashboard.

![image](./tutorial/images/db-pending.png)

The status will change to `Active` when the database is ready, this will only take 2-3 minutes. You will also receive an email when it is ready.

[🏠 Back to Table of Contents](#table-of-contents)


## 2. Security token

✅  **Step 2a:**  [Create a token for your app](https://docs.datastax.com/en/astra/docs/manage-application-tokens.html) to use in the settings screen. Use "Database Administrator" permission.

✅  **Step 2b:**  Copy the token value (eg `AstraCS:KDfdKeNREyWQvDpDrBqwBsUB:ec80667c....`) in your clipboard and save the CSV, this value would not be provided afterward.

**👁️ Expected output**
- <details><summary>Show me!</summary>
    <img src="img/astra-create-token.gif?raw=true" />
</details>

[🏠 Back to Table of Contents](#table-of-contents)

## 3. Table **genre** with GraphQL

✅  **Step 3a:** Open **GraphQL Playground** by 
1. Click on your active database
2. Click `Connect` TAB
3. Click `GRAPHQL API`
4. Clik link to you playground.

*As show on the picture below.*
![image](img/open-playground.png?raw=true)

> *Note that values in the picture do no reflect the database name `netflix_workshop_db`, reason is we do not reproduce every pictures each time*

✅  **Step 3b:** In GraphQL Playground, **Populate HTTP HEADER** variable `x-cassandra-token` on the bottom of the page with your token as shown below

![image](img/graphql-playground.png?raw=true)

✅  **Step 3c:** In GraphQL Playground, create a table with the following mutation, making sure to replace `netflix_keyspace` if you used a different name:

- Copy the following mutation on the left panel
```yaml
mutation {
  reference_list: createTable(
    keyspaceName:"netflix_keyspace",
    tableName:"reference_list",
    ifNotExists:true
    partitionKeys: [ 
      { name: "label", type: {basic: TEXT} }
    ]
    clusteringKeys: [
      { name: "value", type: {basic: TEXT}, order: "ASC" }
    ]
  )
}
```
* Use the arrow in the middle of the screen to execute the query

![image](tutorial/images/playground-1.png?raw=true)

[🏠 Back to Table of Contents](#table-of-contents)

## 4. Insert data in the Table with GraphQL

✅  **Step 4a:** In graphQL playground, change tab to now use `graphql`. Edit the end of the URl to change from `system` to the name of your keyspace: `netflix_keyspace`

✅  **Step 3b:** Populate **HTTP HEADER** variable `x-cassandra-token` on the bottom of the page with your token as shown below (again !! yes this is not the same tab)

![image](img/graphql-playground-2.png?raw=true)

✅  **Step 4c:** In GraphQL Playground,populate the `reference_list` table with the following values

- Copy the following mutation on the left panel

```yaml
mutation insertGenres {
  action: insertreference_list(value: {label:"genre", value:"Action"}) {
    value{value}
  }
  anime: insertreference_list(value: {label:"genre", value:"Anime"}) {
     value{value}
  }
  award: insertreference_list(value: {label:"genre", value:"Award-Winning"}) {
     value{value}
  }
  children: insertreference_list(value: {label:"genre", value:"Children & Family"}) {
     value{value}
  }
  comedies: insertreference_list(value: {label:"genre", value:"Comedies"}) {
     value{value}
  }
  documentaries: insertreference_list(value: {label:"genre", value:"Documentaries"}) {
     value{value}
  }
  drama: insertreference_list(value: {label:"genre", value:"Dramas"}) {
     value{value}
  }
  fantasy: insertreference_list(value: {label:"genre", value:"Fantasy"}) {
     value{value}
  }
  french: insertreference_list(value: {label:"genre", value:"French"}) {
     value{value}
  }
  horror: insertreference_list(value: {label:"genre", value:"Horror"}) {
     value{value}
  }
  independent: insertreference_list(value: {label:"genre", value:"Independent"}) {
     value{value}
  }
  music: insertreference_list(value: {label:"genre", value:"Music & Musicals"}) {
     value{value}
  }
  romance: insertreference_list(value: {label:"genre", value:"Romance"}) {
     value{value}
  }
  scifi: insertreference_list(value: {label:"genre", value:"Sci-Fi"}) {
     value{value}
  }
  thriller: insertreference_list(value: {label:"genre", value:"Thriller"}) {
     value{value}
  }  
}
```

* Use the arrow in the middle of the screen to execute the query

[🏠 Back to Table of Contents](#table-of-contents)

## 5. Retrieving list of values

✅  **Step 5a:** In GraphQL Playground, not changing tab (yeah) list values from the table with the following query.

```yaml
query getAllGenre {
    reference_list (value: {label:"genre"}) {
      values {
      	value
      }
    }
}
```

*👁️ Expected output*
![image](img/graphql-playground-3.png?raw=true)

[🏠 Back to Table of Contents](#table-of-contents)

## 6. Creating a Movies Table

✅  **Step 6a:** Move to tab `GRAPHQL-SCHEMA`, everything should be set, use the following mutation to create a new table:
_Remember to change the keyspaceName if you used something different.

```yaml
mutation {
  movies_by_genre: createTable(
    keyspaceName:"netflix_keyspace",
    tableName:"movies_by_genre",
    ifNotExists: true,
    partitionKeys: [
      { name: "genre", type: {basic: TEXT} }
    ]
    clusteringKeys: [ 
      { name: "year", type: {basic: INT}, order: "DESC" },
      { name: "title", type: {basic: TEXT}, order: "ASC" }
    ]
    values: [
      { name: "synopsis", type: {basic: TEXT} },
      { name: "duration", type: {basic: INT} },
      { name: "thumbnail", type: {basic: TEXT} }
    ]
  )
}
```

*👁️ Expected output*
![image](img/graphql-playground-4.png?raw=true)

[🏠 Back to Table of Contents](#table-of-contents)

## 7. Insert Values in Movie table

✅  **Step 7a:** Mutation to populate movies table: 

```yaml
mutation insertMovies {
  inception: insertmovies_by_genre(
    value: { 
      genre:"Sci-Fi", 
      year:2010,
      title:"Inception",
      synopsis:"Cobb steals information from his targets by entering their dreams.",
      duration:121,
      thumbnail:"https://i.imgur.com/RPa4UdO.mp4"}) {
    value{title}
    }
  
  prometheus: insertmovies_by_genre(value: { 
      genre:"Sci-Fi", 
      year:2012,
      title:"Prometheus",
      synopsis:"After a clue to mankind's origins is discovered, explorers are sent to the darkest corner of the universe.",
      duration:134,
      thumbnail:"https://i.imgur.com/L8k6Bau.mp4"}) {
    value{title}
    }
  
  	aliens: insertmovies_by_genre(value: { 
      genre:"Sci-Fi", 
      year:1986,
      title:"Aliens",
      synopsis:"Ellen Ripley is sent back to the planet LV-426 to establish contact with a terraforming colony.",
      duration:134,
      thumbnail:"https://i.imgur.com/QvkrnyZ.mp4"}) {
    value{title}
    }
  
    bladeRunner: insertmovies_by_genre(value: { 
      genre:"Sci-Fi", 
      year:1982,
      title:"Blade Runner",
      synopsis:"Young Blade Runner K's discovery of a long-buried secret leads him to track down former Blade Runner Rick Deckard.",
      duration:145,
      thumbnail:"https://i.imgur.com/xhhvmj1.mp4"}) {
    value{title}
    }
  }
```

*👁️ Expected output*
![image](img/graphql-playground-5.png?raw=true)

> ℹ️ You can find more movie data in the `data` folder, however, we will be doing a bulk import of all this data shortly.

[🏠 Back to Table of Contents](#table-of-contents)

## 8. Retrieve values from Movie tables

✅  **Step 8a:

```yaml
query getMovieAction {
    movies_by_genre (
      value: {genre:"Sci-Fi"},
       orderBy: [year_DESC]) {
      values {
      	year,
        title,
        duration,
        synopsis,
        thumbnail
      }
    }
}
```

*👁️ Expected output*
![image](img/graphql-playground-6.png?raw=true)

✅ **Step 8b Enable paging:`

```yaml
query getMovieAction {
    movies_by_genre (
      value: {genre:"Sci-Fi"},
       options: {pageSize: 2},
       orderBy: [year_DESC]) {
      values {
      	year,
        title,
        duration,
        synopsis,
        thumbnail
      }
    pageState
    }
}
```

*👁️ Expected output*

![image](tutorial/images/playground-2.png?raw=true)

✅ **Step 8c: Fetch next page paging:

```yaml
query getMovieAction {
    movies_by_genre (
      value: {genre:"Sci-Fi"},
       options: {pageSize: 2, pageState: "<YOUR_PAGE_STATE>"},
       orderBy: [year_DESC]) {
      values {
      	year,
        title,
        duration,
        synopsis,
        thumbnail
      }
    pageState
    }
}
```
 
*👁️ Expected output*

![image](tutorial/images/playground-3.png?raw=true)

[🏠 Back to Table of Contents](#table-of-contents)

## 9. Load a CSV DataSet

✅ **Step 9a: Download the dataset**

Download the dataset

> * ile opens `movies.csv`. This is important as the filename will be the table name.*

<p align="left">
<a href="https://raw.githubusercontent.com/datastaxdevs/workshop-graphql-netflix/main/data/movies.csv">
 <img src="https://dabuttonfactory.com/button.png?t=Download Dataset&f=Roboto-Bold&ts=20&tc=fff&hp=20&vp=15&c=11&bgt=unicolored&bgc=15d798" />
</a>
</p>

✅ **Step 9b: Opening Astra Data Loader Importer**

- Locateing the `Upload Data button` to open the Data Loader.

![image](tutorial/images/import-movies-0.png?raw=true)

✅ **Step 9c: Uploading the dataset**

Drag n drop a single file and look for the file `movies.csv` on your machine, this file has been downloaded in step 9b.

![image](tutorial/images/import-movies-1.png?raw=true)

`Upload Successful`
✅ **Step 9d: Defining target table**

- Locating the field Table Name and setting value to `movies_by_genre`

![image](tutorial/images/import-movies-2.png?raw=true)

- In `Keys and Clustering` -> `genre` as the partition key.

![image](tutorial/images/import-movies-4.png?raw=true)

 `NEXT` to move forward.

✅ **Step 9e: Defining target database**

![image](tutorial/images/import-movies-3.png?raw=true)

| Field | Value |
| --- | --- |
| **Target Database** | `netflix_workshop_db` |
| **Target Keyspace** | `netflix_keyspace` |

 launching the treatment asynchronously.

✅ **Step 9f: Waiting for the batch to import data**


![image](tutorial/images/import-movies-5.png?raw=true)

As you can see the operation here is asynchronous. 

![image](tutorial/images/import-movies-6.png?raw=true)

**The Database is SET !!!**

[🏠 Back to Table of Contents](#table-of-contents)

# Part 2 - Deploy to Production

## 1. Deploying to Netlify

- <details><summary> What does the netlify deploy button do?</summary>The Netlify deploy button will:<ul>
    <li>Create a new repository for you on Github</li>
    <li>Create a site on Netlify</li>
    <li>Link the two together.</li></ul>
</details>

- Clicking the button to deploy

  [![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/datastaxdevs/workshop-graphql-netflix)
 * <details><summary>Show me!</summary>
    <img src="tutorial/images/deploy-to-netlify.gif?raw=true" />
    </details>

This will take a few minutes.

  * `Site deploy in progress` within the Netlify UI, 
    <details>
    <summary>Show me! </summary>
    <img src="tutorial/images/deploy-1.png" />
    </details>

  *  Deploying link to see the build process.
    <details>
    <summary>Show me! </summary>
    <img src="tutorial/images/deploy-2.png" />
    </details>

  *  `Netlify Build Complete`,  **Pushing to repository** 
    <details>
    <summary>Show me! </summary>
    <img src="tutorial/images/deploy-3.png" />
    </details>

  
    <details>
    <summary>Show me! </summary>
    <img src="tutorial/images/deploy-4.png" />
    </details>

## 2. Cloning GitHub repository

    <details>
    <summary>Show me! </summary>
    <img src="tutorial/images/deploy-5.png" />
    </details>

## 3. Launching GitPod IDE

* _Supported by <img src="tutorial/images/chrome-logo.svg" height="20"/> Chrome and <img src="tutorial/images/firefox-logo.svg" height="20"/> Firefox_


![correct notcorrect](tutorial/images/correct-not-correct.png?raw=true)


 * Creating the button
 * <details>
     <summary>CLICK HERE to launch GitPod</summary>

     [![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/from-referrer/)
   </details>
   
ℹ️ _It may take a moment for GitPod to fully initialize.

## 4. Install the Netlify CLI (Command Line Interface)

 ```
 npm install -g netlify-cli
```

 * <details><summary>Show me!</summary>
    <img src="tutorial/images/netlify-install-cli.png?raw=true" />
    </details>

## 5. Retrieving application token to securely connect to the database

Using the token previously generated.

You will also need the GraphQL Endpoint for your keyspace.
First, go to the AstraDB connect page for your database.
![graphql-endpoint-1](tutorial/images/graphql-keyspace-url-01.png)
Then scroll down to find the endpoint for your keyspace.
![graphql-endpoint-1](tutorial/images/graphql-keyspace-url-02.png)

## 6. Configuring Environment Variables and Install Dependencies

✅ Create `.env` file

```ini
ASTRA_DB_APPLICATION_TOKEN={ your_token }
ASTRA_GRAPHQL_ENDPOINT={ your_endpoint }
```

👩‍💻  Installing all the packages

```bash
npm install
```

## 7. Launching the app
  * Run the application 
  ```
  netlify dev
  ```
  * The application should automatically launch in the GitPod preview pane

## 8. Connecting Netlify to the site

Executing each of the commands below.

✅ **Step 8a:**  we'll need to **STOP** the `netlify dev` command we issued a moment ago. In the terminal, `CTRL+C` (control key + the C key) in order to stop the process.

✅ **Step 8b:** Entering the followiing command to pop up a browser to authenticate with netlify

  ```
  netlify login
  ```

*👁️ Expected output*
```
Opening https://app.netlify.com/authorize?....
⠋ Waiting for authorization...^C
```

✅ **Step 8c:** Opening the link in a new WINDOW for the link to work, and authorize Netlify CLi to access Netlify on your behalf.

  > When using GitPod the preview pane **will not display this properly.** You must click the "open in a new window" button in the very top right of the preview pane._

*👁️ Expected output*

```
You are now logged into your Netlify account!
Run netlify status for account details
To see all available commands run: netlify help
gitpod /workspace/workshop-graphql-netflix $ 
```

✅ **Step 8d:** link your workspace to the associated site with the following command

```
netlify link
```

*👁️ Expected output*

![image](tutorial/images/netlify-link.png?raw=true)

✅ **Step 8e:** taking the .env file and uploading it to netlify
  
  ```
  netlify env:import .env
  ```

<!--
  * Will be used to allow you to execute `netlify open`
  ```
  netlify sites:list
  ```
-->

## 9. Deploy to production
Now that you've hooked everything up, time to deplpoy to production.

  * Run
  ```
  netlify build
  ```

  * Then run
  ```
  netlify deploy --prod
  ```

  * Then finally run
  ```
  netlify open:site
  ```
  
  Deployed app to Netlify!
  ![Netlify Setup Example](./tutorial/images/prodDeploy.png?raw=true)


# ALL DONE!

