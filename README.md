### ![GA](https://cloud.githubusercontent.com/assets/40461/8183776/469f976e-1432-11e5-8199-6ac91363302b.png)
### General Assembly, Software Engineering Immersive
# Friday Night Films

by [Vesna Zivanovic](https://github.com/ZVesna) and [Emily Kulesa](https://github.com/emilieeileen)


## Overview

The assignment was to create a **React application** that consumes a **public API**, as part of the **48-hour hackathon**. The project was to be completed in a **team of 2**.

Making use of the The Movie Database (TMDb) public API, we built a website **Friday Night Films** that allows a user to see information about each individual film, get a list of similar films and use the navigation bar to search films by genre or movie title.

<img align = 'center' src='https://i.imgur.com/veLjEoK.png' >

The project was pair-programmed using VSCode Live Share and the driver/navigator technique, to take turns in each role. Website was built using React, JavaScript, HTML and CSS.

You can launch the site on GitHub pages [here](https://zvesna.github.io/project-2/), or find the GitHub repo [here](https://github.com/ZVesna/project-2).


## Brief

Requirements:

* **Consume a public API** - this could be anything but it must make sense for your project
* **Have several components** - At least one classical and one functional
* **The app should include a router** - with several "pages"
* Have **semantically clean HTML** - you make sure you write HTML that makes structural sense rather than thinking about how it might look, which is the job of CSS
* **Be deployed online** and accessible to the public.

## Technologies Used

- HTML5
- JavaScript (ES6)
- SASS
- React
- React Router
- Git
- GitHub
- Insomnia
- Axios
- Semantic React UI


## Approach

### Planning

We spent some time discussing what would be the best topic for this project, given a 48-hour deadline to deliver a fully functional website. Choosing the right API was clearly key to the success of the project and, while we wanted to make something fun and useful, it was important to expand our knowledge of React, React Router and APIs.

Shortly, we agreed that it was a pretty safe option to proceed with a film API, as there were many accessible and well documented APIs, and we could also be creative with visuals.

Once we had selected an API and gathered more information on provided endpoints, we planned out the flow of our app. As one of the requirements was that it should have several pages, we made an initial plan of what these pages should be.


### Structure

Using React Router we have created a number of pathways. Our App.js file:

```
const App = () => (
  <BrowserRouter>
    <NavBar />
    <Switch>
      <Route exact path="/project-2" component={Home}/>
      <Route path="/project-2/genres/" component={Genres}/>
      <Route path="/project-2/search/" component={Search}/>
      <Route path="/project-2/similarfilms/:id" component={Similar}/> 
      <Route path="/project-2/movie/:id" component={Movie}/>
    </Switch>
  </BrowserRouter>
)
	
export default App
```

Having all the paths visible in one place really helped in terms of organizing our time and dividing the workload between the two of us.


### Functionality

In order for content to be shown nicely, films were displayed in a form of a grid. Users could click on a film poster to see the information about each individual film, and 'Similar Films' option to display films that are alike.

<img align = 'center' src='https://i.imgur.com/tTKmHkP.png' >

Users could also use the navigation bar to search films by genre or movie title. We wanted to provide flexibility to search films by multiple means, while creating a clean interface to display selected films.


### Homepage

The homepage was where the users would make the initial decision about what they wanted to look for. We kept it very simple and populated with 20 trending movies. In order to fill the page with content and allow the user to easily navigate to the currently most popular movies, we used **Trending endpoint**.

```js
const Home = () => {
  const [trending, updateTrending] = useState([])
  const [loading, updateLoading] = useState(true)

  useEffect(() => {
    axios.get(`https://api.themoviedb.org/3/trending/movie/day?api_key=${process.env.API_KEY}`)
      .then(({ data }) => {
        updateTrending(data.results)
        updateLoading(false)
      })
  }, [])

  if (loading){
    return <>
      <img src='https://i.imgur.com/jKTJEFh.png'/>
      <h1>Loading films...</h1>
    </>
  }

  return <div className='homepage'> 
    <h1 className='title'>Friday Night Films</h1>
    <div className='trendingdiv'>
      {trending.map((movie) => { 
        return <Link key={movie.id} to={`/project-2/movie/${movie.id}`}>
          <div className='trendingcard'>
            <img className='trendingposter'src={`https://image.tmdb.org/t/p/w154${movie.poster_path}`} alt="Coming soon"/>
          </div>
        </Link>
      })
      }
    </ div>
  </div>
}
```


### Genres Page

While TMDB has many endpoints to filter films, we decided to focus on two categories, genres and keywords. Originally, we attempted to have both of these filters on one Search page, but given the nature of the endpoints, we separated these functionalities into two pages. 

In order to filter the movies based on genre, we created a dropdown menu which lists all the genres, and from there the users could choose the one they prefer. Each genre had a numerical value, which corresponded to the ID number given by the database.

<img align = 'center' src='https://i.imgur.com/eNc96NS.jpg' >

When the page first loads, fetch request from the basic **Discover endpoint** would return films by popularity ranking. As the user selects a genre, the APIUrl changes, as well as the genre value, so only films from that genre are shown.

``` js
export default function Genre({ match }) {
  const id = match.params.id
  const [discoverMovies, updateDiscoverMovies] = useState([])
  const [loading, updateLoading] = useState(true)
  const [genre, updateGenre] = useState('All Genres')
  const [activePage, setActivePage] = useState('')
  const [apiUrl, setApiUrl] = useState(`https://api.themoviedb.org/3/discover/movie?api_key=${process.env.API_KEY}&language=en-US&sort_by=popularity.desc&include_adult=false&include_video=false&page=1`)
  const [pageNo, updatePageNo] = useState('1')

  useEffect(() => {
    axios.get(apiUrl)
      .then(({ data }) => {
        updateDiscoverMovies(data.results)
        updateLoading(false)
      })
  }, [apiUrl])
  
  useEffect(() => {
    axios.get(apiUrl)
      .then(({ data }) => {
        updateDiscoverMovies(data.results)
        updateLoading(false)
      })
  }, [genre])

  if (loading) {
    return <>
      <img src='https://i.imgur.com/jKTJEFh.png'/>
      <h1>Loading films...</h1>
    </>
  }
```


### Search Page

Search by the keyword is enabled using the search bar.

<img align = 'center' src='https://i.imgur.com/CJEQGMg.png' >

The initial logic for this page remained similar to the genre page. Fetch request would return films ranked by popularity. When the user enters the keyword, APIUrl gets updated, and a matching set of films is displayed.

``` js
export default function Search({ match }) {
  const id = match.params.id
  const [discoverMovies, updateDiscoverMovies] = useState([])
  const [loading, updateLoading] = useState(true)
  const [search, updateSearch] = useState('')
  const [activePage, setActivePage] = useState('')
  const [apiUrl, setApiUrl] = useState(`https://api.themoviedb.org/3/discover/movie?api_key=${process.env.API_KEY}&language=en-US&sort_by=popularity.desc&include_adult=false&include_video=false&page=1`)
  const [pageNo, updatePageNo] = useState('1')

  useEffect(() => {
    axios.get(apiUrl)
      .then(({ data }) => {
        updateDiscoverMovies(data.results)
        updateLoading(false)
      })
  }, [apiUrl])

  if (loading) {
    return <>
      <img src='https://i.imgur.com/jKTJEFh.png'/>
      <h1>Loading films...</h1>
    </>
  }
```


### Pagination

Both, the genre and the search page, use pagination. As the API allowed only 20 items to be called at a given time, the pagination gave us an option to call a new request on each page.

The pagination component was placed underneath the displayed items, allowing users to click through pages while still being able to view the 20 results per page.

An example of how the pagination would behave if the user clicked on page number 5:

<img align = 'center' src='https://i.imgur.com/9qbq1sg.png' >

We created a *pageChange* constant variable which determined how the APIUrl would be set, based on the active page.

```js
const pageChange = (page, pageInfo) => {
    setActivePage(pageInfo.activePage)
    if (search === '') {
      setApiUrl(`https://api.themoviedb.org/3/discover/movie?api_key=${process.env.API_KEY}&language=en-US&sort_by=popularity.desc&include_adult=false&include_video=false&page=${pageInfo.activePage}`)
    } else {
      setApiUrl(`https://api.themoviedb.org/3/search/movie?api_key=${process.env.API_KEY}&language=en-US&query=${search}&page=${pageInfo.activePage}&include_adult=false`)
    }
    updatePageNo(pageInfo.activePage)
  }
```

The idea for simplistic design was taken from Semantic React UI, and it got along well with our art deco theme.


### Single Movie and Similar Films

We used CSS Flexbox in order to design responsive layout structure for a single movie page. It was important to simultaneously highlight featured movie posters, while displaying the relevant information in a clear way.

As there was a **Similar Films endpoint** provided from the API, we thought it was a good idea to include an additional component into the project. This option was added on the single movie page, where users could click and get a list of similar films to the one they selected.


### Styling

- Going through some examples of movie database websites, we have noticed that many had similar layouts and color themes. This observation led to the decision to create something that visually stood out. Using a color palette designed from an old Hollywood theatre, we decided on an Art Deco, Golden Age of Hollywood design for our website. The color palette also proved useful as the dark blue, light teal and gold tones matched well as a background for many of the movie posters on the database. We also found the PoiretOne font fit in perfectly with our theme, giving our website a vintage, classic feel, while showing the latest modern films.

- While The Movie Database has many items fully fleshed out, some films may not have similar films or a movie poster available. To keep styling consistent, we created a placeholder poster to fill the movie poster space and a custom div that displayed if no similar films could be found.

- We also added a loading image when it takes longer for content to appear on the page.

## Conclusion

**Wins**

- We both really enjoyed the paired programming experience. Since this project was only 48 hours, we divided up the work and helped each other when we got stuck. This allowed us to work to our strengths, splitting up work based on interests. We were both very proud of the result.

**Potential future features**

- If we were to approach this project again, we may look at a styling framework, such as Bulma or Bootstrap. We chose not to use one as neither one of us were fully comfortable at the time, but given our exposure to them in subsequent projects, it would present a new and interesting challenge. 

**Bugs**

- When on the genres drop down, if you select one genre and then return to All Genres, the films displayed remain unchanged.

**Lessons learned**

- Good communication is the key to the successful short-term project.

- It was a great learning experience in terms of applied skills, teamwork, organization and time management.


## Credit

API courtesy of [The Movie Database (TMDb)](https://www.themoviedb.org/).