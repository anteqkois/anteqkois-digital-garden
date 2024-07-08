---
title: Cancel pending axios request before making new to the same url
tags:
  - dev
  - axios
  - optymalization
---

## VueJS

Simple vueJS hook to handle request cancelation when new config was provided

```
import axios, { AxiosRequestConfig, AxiosResponse } from 'axios';
import { ref } from 'vue';

export const useAxios = () => {
  // store tokens here to prevent re-usage token for requests from two different places
  const requestAbortController = ref<AbortController>();

  const withCancelation = <T>(
    config: AxiosRequestConfig<T>
  ): Promise<AxiosResponse<T>> => {
    if (requestAbortController.value)
      requestAbortController.value.abort(`Cancel due to re-request data`);

    requestAbortController.value = new AbortController();

    return axios({ ...config, signal: requestAbortController.value.signal });
  };

  return { withCancelation };
};
```

Then simple call inside your bussines logic

```
const { data } = await withCancelation<{
  data: DataResource[];
  cursor: string;
}>({
  method: 'GET',
  url: `${API_URL}/data-aggregator`,
  params: {
    ...filter,
    limit: limit.value,
    cursor: cursor.value,
    lang: locale.value,
  },
});
```
