# Federated Search

Federated search is the practice of taking one query in your frontend and sending it to multiple services and collecting the results.

If we wanted to implement a federated search in Next.js/React we could do something like this example.

1. In the frontend we would have a search that collects the query and sends it to our server via an API endpoint:
```
import { useState } from 'react';

const SearchForm = () => {
  const [query, setQuery] = useState('');

  const handleSubmit = async (event) => {
    event.preventDefault();

    // Send the query to the server here
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        placeholder="Search"
        value={query}
        onChange={(event) => setQuery(event.target.value)}
      />
      <button type="submit">Search</button>
    </form>
  );
};
```

2. On the server we'd have an endpoint that accepts the query and send it to each of the different services using their APIs. The results would be collected and returned to the app frontend:
```
// server/api/search.js
import { queryGitHub, queryNotion, queryOpenMetadata } from './services';

export default async (req, res) => {
  const { query } = req.body;

  // Query each service and collect the results
  const gitHubResults = await queryGitHub(query);
  const notionResults = await queryNotion(query);
  const openMetadataResults = await queryOpenMetadata(query);

  // Return the results to the frontend
  res.json({
    gitHubResults,
    notionResults,
    openMetadataResults,
  });
};
```

3. Then on the frontend we would handle the respone by displaying the results in the UI:
```
const SearchResults = ({ results }) => {
  return (
    <ul>
      {results.map((result) => (
        <li key={result.id}>{result.title}</li>
      ))}
    </ul>
  );
};

const SearchPage = () => {
  const [results, setResults] = useState([]);

  const handleSubmit = async (event) => {
    event.preventDefault();

    const res = await fetch('/api/search', {
      method: 'POST',
      body: JSON.stringify({ query }),
    });
    const data = await res.json();

    // Set the results in the state
    setResults(data);
  };

  return (
    <>
      <SearchForm onSubmit={handleSubmit} />
      <SearchResults results={results} />
    </>
  );
};
```