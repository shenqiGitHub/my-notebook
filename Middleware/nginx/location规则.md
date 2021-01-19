

#### 1. Location 的判断顺序

```mermaid
graph TD
A(URI) --> B{是否精准匹配}
B --> |No| C[普通匹配1<br> 普通匹配2<br>...<br>普通匹配n ]
B --> |Yes| D([返回精准匹配结果\,并结束])
C --> E[正则匹配1<br>正则匹配2<br>...<br>正则匹配n]
E --> |按照正则表达式顺序为准| F([如果匹配到直接立即返回])
C --> |Yes| H{是否命中多个}
H --> |Yes| H1([匹配最长的结果])
H --> |No| H3([记忆匹配结果])
```







#### 2. nginx reverse proxy 斜杠产生的作用

```shell
location /some/path/ {
	proxy_pass http://www.example.com/link/;
}
```



Note that in the first example above, the address of the proxied server is followed by a URI, `/link/`. If the URI is specified along with the address, ==it replaces the part of the request URI that matches the location parameter==. For example, here the request with the `/some/path/page.html` URI will be proxied to `http://www.example.com/link/page.html`. If the address is specified without a URI, or it is not possible to determine the part of URI to be replaced, the full request URI is passed (possibly, modified).