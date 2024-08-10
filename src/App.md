# USEPOPCORN NOTES

## 1. Average
Nantinya digunakan untuk menghitung `avgImdbRating`, `avgUserRating`, `avgRuntime` di component `WatchedSummary`.

```js
const average = (arr) =>
  arr.reduce((acc, cur, i, arr) => acc + cur / arr.length, 0);
```

Misal :

```js
let arr = [10, 20, 30];
```
Maka $\frac{10}{3}$ + $\frac{20}{3}$ + $\frac{30}{3}$ = 20 

## 2. State di Component App
```js
//untuk search film.
const [query, setQuery] = useState(""); 
//untuk menyimpan film yang telah disearch. 
const [movies, setMovies] = useState([]); 
//untuk mengeset film ke kategori yang telah ditonton.
const [watched, setWatched] = useState([]); 
//untuk mengeset perlu tulisan loading atau tidak.
const [isLoading, setIsLoading] = useState(false);
//untuk mengeset pesan error
const [error, setError] = useState("");
//untuk mengeset film yang telah di select dan akan ditampilkan ke box kanan.
const [selectedId, setSelectedId] = useState(null);
```

## Fungsi di Component App

1.
    ```js
    function handleSelectMovie(id) { // fungsi select movie
        setSelectedId((selectedId) => (id === selectedId ? null : id));
    }
    ```
    Fungsi diatas memakai arrow function.<br>
    akan mengembalikan nilai null jika yang di klik film yang sama. dan akan mengembalikan nilai id baru klik film baru.

    contoh lain arrow function : 
    ```js
    const add = (a, b) => a + b;
    ```

## useEffect di Component App

``` js
useEffect(
    function () {
      const controller = new AbortController(); //supaya menghapus data fetched yang telah diunduh tapi tidak digunakan

      async function fetchMovies() {
        try {
          setIsLoading(true);
          setError("");

          const res = await fetch(
            `http://www.omdbapi.com/?apikey=${KEY}&s=${query}`,
            { signal: controller.signal } //supaya menghapus data fetched yang telah diunduh tapi tidak digunakan
          );

          if (!res.ok) 
          //jika Anda menggunakan Fetch API di JavaScript, properti res.ok 
          //akan selalu ada di objek Response yang dihasilkan oleh fetch()
            throw new Error("Something went wrong with fetching movies");

          const data = await res.json();
          if (data.Response === "False") throw new Error("Movie not found"); 
          //Kalau data.Response === "False" ini harus baca dokumentasi API

          setMovies(data.Search);
          setError("");
        } catch (err) {
          if (err.name !== "AbortError") {
            console.log(err.message);
            setError(err.message);
          }
        } finally {
          setIsLoading(false);
        }
      }

      if (query.length < 3) {
        setMovies([]);
        setError("");
        return;
      }

      handleCloseMovie(); /*ini karena jika querynya diganti (menulis 
      di serach), maka box kanan (detail movie), akan ditutup */  
      fetchMovies();

      return function () { //Kenapa harus pakai return function?
        controller.abort();//supaya menghapus data fetched yang telah diunduh tapi tidak digunakan
      };
    },
    [query]
  );
```

1. async dan await adalah pasangan yang saling terkait dalam JavaScript untuk menangani operasi asinkron dengan cara yang lebih bersih dan mudah dibaca. jadi jika dalam fungsi ada awaitnya, maka sebelum functionnya harus ditulis `async`


2. `await res.json()` digunakan untuk mengonversi respons dari server (yang berbentuk Response dari fetch) menjadi objek JavaScript.

3. ```js
    if (err.name !== "AbortError") {
            console.log(err.message);
            setError(err.message);
          }
    ``` 
    Ketika permintaan dibatalkan menggunakan AbortController, fetch akan melemparkan error dengan name yang setara dengan "AbortError". <br>
    Jadi jika `err.name == "AbortError"`maka tidak perlu ditanggapi

4. ```js
    if (query.length < 3) {
            setMovies([]);
            setError("");
            return;
        }
    ```
    ### Kenapa ada `return` ?
    Penghentian Eksekusi: return digunakan di sini untuk menghentikan eksekusi kode di bawahnya dalam fungsi useEffect. Ini memastikan bahwa fungsi fetchMovies tidak dipanggil jika query tidak memenuhi syarat. Tanpa return, fetchMovies akan dipanggil bahkan jika query tidak valid, yang mengakibatkan permintaan yang tidak perlu atau tidak valid.

5.  ```js
        return function () { 
        controller.abort();//supaya menghapus data fetched yang telah diunduh tapi tidak digunakan
      };
    ``` 
    `return function` di dalam useEffect memang khusus digunakan untuk mendefinisikan clean up function. jadi, return function hanya khusus untuk cleanup function.

    <div style="text-align: justify">
    Jika langsung menulis `controller.abort();` tanpa menggunakan return function di dalam useEffect akan menyebabkan perintah controller.abort(); dijalankan segera setelah fetch request dimulai. Ini bukan yang Anda inginkan, karena Anda ingin controller.abort(); hanya dijalankan ketika komponen di-unmount atau sebelum useEffect dijalankan kembali (misalnya, ketika dependencies berubah).
    <div>


## Bagian JSX Component App

### Bagian Navbar
```js
<NavBar>
        <Search query={query} setQuery={setQuery} />
        <NumResults movies={movies} />
</NavBar> 
```

```js
  function NavBar({ children }) {
  return (
    <nav className="nav-bar">
      <Logo />
      {children}
    </nav>
  );
}

function Logo() {
  return (
    <div className="logo">
      <span role="img">🍿</span>
      <h1>usePopcorn</h1>
    </div>
  );
}

function Search({ query, setQuery }) {
  return (
    <input
      className="search"
      type="text"
      placeholder="Search movies..."
      value={query}
      onChange={(e) => setQuery(e.target.value)}
    />
  );
}

function NumResults({ movies }) {
  return (
    <p className="num-results">
      Found <strong>{movies.length}</strong> results
    </p>
  );
}
```
Jadi Component Logo tidak perlu dijadikan children karena tidak 
memakai props.

## Box bagian kiri

```js
<Box>
          {/* {isLoading ? <Loader /> : <MovieList movies={movies} />} */}
          {isLoading && <Loader />}
          {!isLoading && !error && (
            <MovieList movies={movies} onSelectMovie={handleSelectMovie} />
          )}
          {error && <ErrorMessage message={error} />}
        </Box>
```

- Kalau Loading, maka menampilkan Component Loader <br>
- Kalau tidak Loading dan tidak Error, maka menampilkan MovieList
<br>
- Kalau Error, maka menampilkan pesan Error.

## Box bagian kanan
```js
<Box>
  {selectedId ? (
    <MovieDetails
      selectedId={selectedId}
      onCloseMovie={handleCloseMovie}
      onAddWatched={handleAddWatched}
      watched={watched}
    />
  ) : (
    <>
      <WatchedSummary watched={watched} />
      <WatchedMoviesList
        watched={watched}
        onDeleteWatched={handleDeleteWatched}
      />
    </>
  )}
</Box>
```
- ### Apakah ada selectedId ?
- Kalau IYA, Tampilkan component MovieDetails
- Jika TIDAK, Tampilkan component WatchedSummary dan WatchedMovieList

## <u>component MovieDetails</u>

```js
function MovieDetails({ selectedId, onCloseMovie, onAddWatched, watched }) {
  const [movie, setMovie] = useState({});
  const [isLoading, setIsLoading] = useState(false);
  const [userRating, setUserRating] = useState("");

  const isWatched = watched.map((movie) => movie.imdbID).includes(selectedId);
  const watchedUserRating = watched.find(
    (movie) => movie.imdbID === selectedId
  )?.userRating;

  const {
    Title: title,
    Year: year,
    Poster: poster,
    Runtime: runtime,
    imdbRating,
    Plot: plot,
    Released: released,
    Actors: actors,
    Director: director,
    Genre: genre,
  } = movie;

  function handleAdd() {
    const newWatchedMovie = {
      imdbID: selectedId,
      title,
      year,
      poster,
      imdbRating: Number(imdbRating),
      runtime: Number(runtime.split(" ").at(0)),
      userRating,
    };

    onAddWatched(newWatchedMovie);
    onCloseMovie();
  }

  useEffect(
    function () {
      function callback(e) { // jika buka movie detail untuk close movie ketika pencet tombol esc
        if (e.code === "Escape") {
          onCloseMovie();
          console.log("CLOSING")
        }
      }

      document.addEventListener("keydown", callback);

      
    },
    [onCloseMovie] //kalau onCloseMovie bergerak, maka code yang diatas akan jalan
  );

  useEffect(
    function () {
      async function getMovieDetails() {
        setIsLoading(true);
        const res = await fetch(
          `http://www.omdbapi.com/?apikey=${KEY}&i=${selectedId}`
        );
        const data = await res.json();
        setMovie(data);
        setIsLoading(false);
      }
      getMovieDetails();
    },
    [selectedId]
  );

  useEffect(
    function () {
      if (!title) return;
      document.title = `Movie | ${title}`;

      return function () {
        document.title = "usePopcorn";
        // console.log(`Clean up effect for movie ${title}`);
      };
    },
    [title]
  );

  return (
    <div className="details">
      {isLoading ? (
        <Loader />
      ) : (
        <>
          <header>
            <button className="btn-back" onClick={onCloseMovie}>
              &larr;
            </button>
            <img src={poster} alt={`Poster of ${movie} movie`} />
            <div className="details-overview">
              <h2>{title}</h2>
              <p>
                {released} &bull; {runtime}
              </p>
              <p>{genre}</p>
              <p>
                <span>⭐️</span>
                {imdbRating} IMDb rating
              </p>
            </div>
          </header>
          <section>
            <div className="rating">
              {!isWatched ? (
                <>
                  <StarRating
                    maxRating={10}
                    size={24}
                    onSetRating={setUserRating}
                  />
                  {userRating > 0 && (
                    <button className="btn-add" onClick={handleAdd}>
                      + Add to list
                    </button>
                  )}
                </>
              ) : (
                <p>
                  You rated with movie {watchedUserRating} <span>⭐️</span>
                </p>
              )}
            </div>
            <p>
              <em>{plot}</em>
            </p>
            <p>Starring {actors}</p>
            <p>Directed by {director}</p>
          </section>
        </>
      )}
    </div>
  );
}
```

