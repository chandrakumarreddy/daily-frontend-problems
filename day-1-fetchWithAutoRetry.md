For a web application, fetching API data is a common task.

But the API calls might fail because of Network problems. Usually we could show a screen for Network Error and ask users to retry.

One approach to handle this is auto retry when network error occurs.

You are asked to create a fetchWithAutoRetry(fetcher, count), which automatically fetch again when error happens, until the maximum count is met.

For the problem here, there is no need to detect network error, you can just retry on all promise rejections.

### fetchWithAutoRetry

```
function fetchWithAutoRetry(fetcher, count) {
  return new Promise((resolve, reject) => {
    const attempt = () => {
      fetcher()
        .then(resolve) 
        .catch(() => {
          if (count === 0) {
            reject();
          } else {
            count--;
            attempt();
          }
        });
    };   
    attempt();
  });
}
```

- It takes a fetcher function which returns a promise, and a count of maximum retries
- It returns a new promise
- On promise resolve, it passes the result to the returned promise
- On promise reject, it checks:
- If count is 0, rejects the returned promise
- Otherwise decrements count and calls attempt() again to retry the fetcher

### fetchWithAutoRetry using async-await

```
function fetchWithAutoRetry(fetcher, count, interval = 1000) {
  return new Promise(async (resolve, reject) => {
    const attempt = async () => {
      try {
        const data = await fetcher();
        resolve(data);
      } catch (err) {
        if (count === 0) {
          reject(err);    
        } else {
          count--;
          setTimeout(attempt, interval);
        }
      }  
    }
    attempt();    
  });
}
```

Use async/await for cleaner syntax


# Adding a delay between retries can help if the API failure was due to a temporary issue
