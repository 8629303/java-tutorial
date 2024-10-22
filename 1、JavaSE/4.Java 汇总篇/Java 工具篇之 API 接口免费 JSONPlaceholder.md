# 一、认识 JSONPlaceholder

官网地址：http://jsonplaceholder.typicode.com/

JSONPlaceholder 是一个提在线 REST API 的网站，我们在开发时可以使用它获取一些假数据、假图片。其返回的数据为 JSON 格式，且同时支持 HTTP 和 HTTPS 这两种请求类型。

JSONPlaceholder 一共有6种通用资源（Resources）：

| URI                                                       | Data Size    |
| --------------------------------------------------------- | ------------ |
| [/posts](http://jsonplaceholder.typicode.com/posts)       | 100 posts    |
| [/comments](http://jsonplaceholder.typicode.com/comments) | 500 comments |
| [/albums](http://jsonplaceholder.typicode.com/albums)     | 100 albums   |
| [/photos](http://jsonplaceholder.typicode.com/photos)     | 5000 photos  |
| [/todos](http://jsonplaceholder.typicode.com/todos)       | 200 todos    |
| [/users](http://jsonplaceholder.typicode.com/users)       | 10 users     |

Routes：All HTTP methods are supported. You can use http or https for your requests.

| HTTP Methods | URI                                                          |
| ------------ | ------------------------------------------------------------ |
| GET          | [/posts](http://jsonplaceholder.typicode.com/posts)          |
| GET          | [/posts/1](http://jsonplaceholder.typicode.com/posts/1)      |
| GET          | [/posts/1/comments](http://jsonplaceholder.typicode.com/posts/1/comments) |
| GET          | [/comments?postId=1](http://jsonplaceholder.typicode.com/comments?postId=1) |
| POST         | /posts                                                       |
| PUT          | /posts/1                                                     |
| PATCH        | /posts/1                                                     |
| DELETE       | /posts/1                                                     |

**Note**: see [guide](http://jsonplaceholder.typicode.com/guide) for usage examples.



# 一、接口目录总结

1、帖子接口：

- 获取帖子列表：http://jsonplaceholder.typicode.com/posts
- 根据帖子ID获取详情：https://jsonplaceholder.typicode.com/posts/1
- 获取某个用户所有的帖子：http://jsonplaceholder.typicode.com/posts?userId=5

 

2、评论接口

- 获取评论列表：http://jsonplaceholder.typicode.com/comments
- 获取某个帖子的所有评论（两种方式）：http://jsonplaceholder.typicode.com/comments?postId=4
- 获取某个帖子所有的评论（两种方式）：https://jsonplaceholder.typicode.com/posts/1/comments

 

3、专辑接口：

- 获取专辑列表：http://jsonplaceholder.typicode.com/albums
- 根据专辑ID获取详情：http://jsonplaceholder.typicode.com/albums/6
- 获取某个用户所有专辑：http://jsonplaceholder.typicode.com/albums?userId=9

 

4、待办事宜接口：

- 获取待办事宜列表：http://jsonplaceholder.typicode.com/todos
- 根据待办ID获取详情：http://jsonplaceholder.typicode.com/todos/6
- 获取某个用户所有待办事宜：http://jsonplaceholder.typicode.com/todos?userId=9

 

5、用户接口：

- 获取用户列表：http://jsonplaceholder.typicode.com/users
- 根据用户ID获取详情：http://jsonplaceholder.typicode.com/users/5

 

6、照片接口：

- 获取照片列表：http://jsonplaceholder.typicode.com/photos
- 根据照片ID获取详情：http://jsonplaceholder.typicode.com/photos/8
- 获取某个专辑所有照片：http://jsonplaceholder.typicode.com/photos?albumId=5





# 二、GET 请求数据

PS：GET方式请求可以直接使用浏览器访问。

## 1、获取文章（贴子）数据

（1）使用此地址可以获取到文章列表假数据，共100条，有帖子ID、发贴人ID、标题、内容。

- http://jsonplaceholder.typicode.com/posts

```json
[
    {
        "userId": 1,
        "id": 1,
        "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
        "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
    },
    {
        "userId": 1,
        "id": 2,
        "title": "qui est esse",
        "body": "est rerum tempore vitae\nsequi sint nihil reprehenderit dolor beatae ea dolores neque\nfugiat blanditiis voluptate porro vel nihil molestiae ut reiciendis\nqui aperiam non debitis possimus qui neque nisi nulla"
    },
    {
        "userId": 1,
        "id": 3,
        "title": "ea molestias quasi exercitationem repellat qui ipsa sit aut",
        "body": "et iusto sed quo iure\nvoluptatem occaecati omnis eligendi aut ad\nvoluptatem doloribus vel accusantium quis pariatur\nmolestiae porro eius odio et labore et velit aut"
    },
    {
        "userId": 1,
        "id": 4,
        "title": "eum et est occaecati",
        "body": "ullam et saepe reiciendis voluptatem adipisci\nsit amet autem assumenda provident rerum culpa\nquis hic commodi nesciunt rem tenetur doloremque ipsam iure\nquis sunt voluptatem rerum illo velit"
    },
    {
        "userId": 1,
        "id": 5,
        "title": "nesciunt quas odio",
        "body": "repudiandae veniam quaerat sunt sed\nalias aut fugiat sit autem sed est\nvoluptatem omnis possimus esse voluptatibus quis\nest aut tenetur dolor neque"
    },
    {
        "userId": 1,
        "id": 6,
        "title": "dolorem eum magni eos aperiam quia",
        "body": "ut aspernatur corporis harum nihil quis provident sequi\nmollitia nobis aliquid molestiae\nperspiciatis et ea nemo ab reprehenderit accusantium quas\nvoluptate dolores velit et doloremque molestiae"
    },
    {
        "userId": 1,
        "id": 7,
        "title": "magnam facilis autem",
        "body": "dolore placeat quibusdam ea quo vitae\nmagni quis enim qui quis quo nemo aut saepe\nquidem repellat excepturi ut quia\nsunt ut sequi eos ea sed quas"
    },
    {
        "userId": 1,
        "id": 8,
        "title": "dolorem dolore est ipsam",
        "body": "dignissimos aperiam dolorem qui eum\nfacilis quibusdam animi sint suscipit qui sint possimus cum\nquaerat magni maiores excepturi\nipsam ut commodi dolor voluptatum modi aut vitae"
    },
    {
        "userId": 1,
        "id": 9,
        "title": "nesciunt iure omnis dolorem tempora et accusantium",
        "body": "consectetur animi nesciunt iure dolore\nenim quia ad\nveniam autem ut quam aut nobis\net est aut quod aut provident voluptas autem voluptas"
    },
    {
        "userId": 1,
        "id": 10,
        "title": "optio molestias id quia eum",
        "body": "quo et expedita modi cum officia vel magni\ndoloribus qui repudiandae\nvero nisi sit\nquos veniam quod sed accusamus veritatis error"
    },
    // 省略了中间的数据
    {
        "userId": 10,
        "id": 91,
        "title": "aut amet sed",
        "body": "libero voluptate eveniet aperiam sed\nsunt placeat suscipit molestias\nsimilique fugit nam natus\nexpedita consequatur consequatur dolores quia eos et placeat"
    },
    {
        "userId": 10,
        "id": 92,
        "title": "ratione ex tenetur perferendis",
        "body": "aut et excepturi dicta laudantium sint rerum nihil\nlaudantium et at\na neque minima officia et similique libero et\ncommodi voluptate qui"
    },
    {
        "userId": 10,
        "id": 93,
        "title": "beatae soluta recusandae",
        "body": "dolorem quibusdam ducimus consequuntur dicta aut quo laboriosam\nvoluptatem quis enim recusandae ut sed sunt\nnostrum est odit totam\nsit error sed sunt eveniet provident qui nulla"
    },
    {
        "userId": 10,
        "id": 94,
        "title": "qui qui voluptates illo iste minima",
        "body": "aspernatur expedita soluta quo ab ut similique\nexpedita dolores amet\nsed temporibus distinctio magnam saepe deleniti\nomnis facilis nam ipsum natus sint similique omnis"
    },
    {
        "userId": 10,
        "id": 95,
        "title": "id minus libero illum nam ad officiis",
        "body": "earum voluptatem facere provident blanditiis velit laboriosam\npariatur accusamus odio saepe\ncumque dolor qui a dicta ab doloribus consequatur omnis\ncorporis cupiditate eaque assumenda ad nesciunt"
    },
    {
        "userId": 10,
        "id": 96,
        "title": "quaerat velit veniam amet cupiditate aut numquam ut sequi",
        "body": "in non odio excepturi sint eum\nlabore voluptates vitae quia qui et\ninventore itaque rerum\nveniam non exercitationem delectus aut"
    },
    {
        "userId": 10,
        "id": 97,
        "title": "quas fugiat ut perspiciatis vero provident",
        "body": "eum non blanditiis soluta porro quibusdam voluptas\nvel voluptatem qui placeat dolores qui velit aut\nvel inventore aut cumque culpa explicabo aliquid at\nperspiciatis est et voluptatem dignissimos dolor itaque sit nam"
    },
    {
        "userId": 10,
        "id": 98,
        "title": "laboriosam dolor voluptates",
        "body": "doloremque ex facilis sit sint culpa\nsoluta assumenda eligendi non ut eius\nsequi ducimus vel quasi\nveritatis est dolores"
    },
    {
        "userId": 10,
        "id": 99,
        "title": "temporibus sit alias delectus eligendi possimus magni",
        "body": "quo deleniti praesentium dicta non quod\naut est molestias\nmolestias et officia quis nihil\nitaque dolorem quia"
    },
    {
        "userId": 10,
        "id": 100,
        "title": "at nam consequatur ea labore ea harum",
        "body": "cupiditate quo est a modi nesciunt soluta\nipsa voluptas error itaque dicta in\nautem qui minus magnam et distinctio eum\naccusamus ratione error aut"
    }
]
```

（2）根据传入的贴子 ID，可以得到具体某个贴子的数据，里面同样包含贴子 ID、发贴人 ID、标题、内容。

- http://jsonplaceholder.typicode.com/posts/55

```json
{
    "userId": 6,
    "id": 55,
    "title": "sit vel voluptatem et non libero",
    "body": "debitis excepturi ea perferendis harum libero optio\neos accusamus cum fuga ut sapiente repudiandae\net ut incidunt omnis molestiae\nnihil ut eum odit"
}
```

（3）根据发贴人 ID 来查询他发的所有贴子，里面同样包含贴子 ID、发贴人 ID、标题、内容。

- http://jsonplaceholder.typicode.com/posts?userId=5

```json
[
    {
        "userId": 5,
        "id": 41,
        "title": "non est facere",
        "body": "molestias id nostrum\nexcepturi molestiae dolore omnis repellendus quaerat saepe\nconsectetur iste quaerat tenetur asperiores accusamus ex ut\nnam quidem est ducimus sunt debitis saepe"
    },
    {
        "userId": 5,
        "id": 42,
        "title": "commodi ullam sint et excepturi error explicabo praesentium voluptas",
        "body": "odio fugit voluptatum ducimus earum autem est incidunt voluptatem\nodit reiciendis aliquam sunt sequi nulla dolorem\nnon facere repellendus voluptates quia\nratione harum vitae ut"
    },
    {
        "userId": 5,
        "id": 43,
        "title": "eligendi iste nostrum consequuntur adipisci praesentium sit beatae perferendis",
        "body": "similique fugit est\nillum et dolorum harum et voluptate eaque quidem\nexercitationem quos nam commodi possimus cum odio nihil nulla\ndolorum exercitationem magnam ex et a et distinctio debitis"
    },
    {
        "userId": 5,
        "id": 44,
        "title": "optio dolor molestias sit",
        "body": "temporibus est consectetur dolore\net libero debitis vel velit laboriosam quia\nipsum quibusdam qui itaque fuga rem aut\nea et iure quam sed maxime ut distinctio quae"
    },
    {
        "userId": 5,
        "id": 45,
        "title": "ut numquam possimus omnis eius suscipit laudantium iure",
        "body": "est natus reiciendis nihil possimus aut provident\nex et dolor\nrepellat pariatur est\nnobis rerum repellendus dolorem autem"
    },
    {
        "userId": 5,
        "id": 46,
        "title": "aut quo modi neque nostrum ducimus",
        "body": "voluptatem quisquam iste\nvoluptatibus natus officiis facilis dolorem\nquis quas ipsam\nvel et voluptatum in aliquid"
    },
    {
        "userId": 5,
        "id": 47,
        "title": "quibusdam cumque rem aut deserunt",
        "body": "voluptatem assumenda ut qui ut cupiditate aut impedit veniam\noccaecati nemo illum voluptatem laudantium\nmolestiae beatae rerum ea iure soluta nostrum\neligendi et voluptate"
    },
    {
        "userId": 5,
        "id": 48,
        "title": "ut voluptatem illum ea doloribus itaque eos",
        "body": "voluptates quo voluptatem facilis iure occaecati\nvel assumenda rerum officia et\nillum perspiciatis ab deleniti\nlaudantium repellat ad ut et autem reprehenderit"
    },
    {
        "userId": 5,
        "id": 49,
        "title": "laborum non sunt aut ut assumenda perspiciatis voluptas",
        "body": "inventore ab sint\nnatus fugit id nulla sequi architecto nihil quaerat\neos tenetur in in eum veritatis non\nquibusdam officiis aspernatur cumque aut commodi aut"
    },
    {
        "userId": 5,
        "id": 50,
        "title": "repellendus qui recusandae incidunt voluptates tenetur qui omnis exercitationem",
        "body": "error suscipit maxime adipisci consequuntur recusandae\nvoluptas eligendi et est et voluptates\nquia distinctio ab amet quaerat molestiae et vitae\nadipisci impedit sequi nesciunt quis consectetur"
    }
]
```



## 2、获取评论数据

（1）获取所有文章的所有评论，共500条。每条内容都包含有贴子 ID、评论 ID、评论人姓名、评论人邮箱、评论内容。

- http://jsonplaceholder.typicode.com/comments

```json
[
    {
        "postId": 1,
        "id": 1,
        "name": "id labore ex et quam laborum",
        "email": "Eliseo@gardner.biz",
        "body": "laudantium enim quasi est quidem magnam voluptate ipsam eos\ntempora quo necessitatibus\ndolor quam autem quasi\nreiciendis et nam sapiente accusantium"
    },
    {
        "postId": 1,
        "id": 2,
        "name": "quo vero reiciendis velit similique earum",
        "email": "Jayne_Kuhic@sydney.com",
        "body": "est natus enim nihil est dolore omnis voluptatem numquam\net omnis occaecati quod ullam at\nvoluptatem error expedita pariatur\nnihil sint nostrum voluptatem reiciendis et"
    },
    {
        "postId": 1,
        "id": 3,
        "name": "odio adipisci rerum aut animi",
        "email": "Nikita@garfield.biz",
        "body": "quia molestiae reprehenderit quasi aspernatur\naut expedita occaecati aliquam eveniet laudantium\nomnis quibusdam delectus saepe quia accusamus maiores nam est\ncum et ducimus et vero voluptates excepturi deleniti ratione"
    },
    {
        "postId": 1,
        "id": 4,
        "name": "alias odio sit",
        "email": "Lew@alysha.tv",
        "body": "non et atque\noccaecati deserunt quas accusantium unde odit nobis qui voluptatem\nquia voluptas consequuntur itaque dolor\net qui rerum deleniti ut occaecati"
    },
    {
        "postId": 1,
        "id": 5,
        "name": "vero eaque aliquid doloribus et culpa",
        "email": "Hayden@althea.biz",
        "body": "harum non quasi et ratione\ntempore iure ex voluptates in ratione\nharum architecto fugit inventore cupiditate\nvoluptates magni quo et"
    },
    {
        "postId": 2,
        "id": 6,
        "name": "et fugit eligendi deleniti quidem qui sint nihil autem",
        "email": "Presley.Mueller@myrl.com",
        "body": "doloribus at sed quis culpa deserunt consectetur qui praesentium\naccusamus fugiat dicta\nvoluptatem rerum ut voluptate autem\nvoluptatem repellendus aspernatur dolorem in"
    },
    {
        "postId": 2,
        "id": 7,
        "name": "repellat consequatur praesentium vel minus molestias voluptatum",
        "email": "Dallas@ole.me",
        "body": "maiores sed dolores similique labore et inventore et\nquasi temporibus esse sunt id et\neos voluptatem aliquam\naliquid ratione corporis molestiae mollitia quia et magnam dolor"
    },
    {
        "postId": 2,
        "id": 8,
        "name": "et omnis dolorem",
        "email": "Mallory_Kunze@marie.org",
        "body": "ut voluptatem corrupti velit\nad voluptatem maiores\net nisi velit vero accusamus maiores\nvoluptates quia aliquid ullam eaque"
    },
    {
        "postId": 2,
        "id": 9,
        "name": "provident id voluptas",
        "email": "Meghan_Littel@rene.us",
        "body": "sapiente assumenda molestiae atque\nadipisci laborum distinctio aperiam et ab ut omnis\net occaecati aspernatur odit sit rem expedita\nquas enim ipsam minus"
    },
    {
        "postId": 2,
        "id": 10,
        "name": "eaque et deleniti atque tenetur ut quo ut",
        "email": "Carmen_Keeling@caroline.name",
        "body": "voluptate iusto quis nobis reprehenderit ipsum amet nulla\nquia quas dolores velit et non\naut quia necessitatibus\nnostrum quaerat nulla et accusamus nisi facilis"
    },
    // 中间省略了许多数据
    {
        "postId": 100,
        "id": 496,
        "name": "et occaecati asperiores quas voluptas ipsam nostrum",
        "email": "Zola@lizzie.com",
        "body": "neque unde voluptatem iure\nodio excepturi ipsam ad id\nipsa sed expedita error quam\nvoluptatem tempora necessitatibus suscipit culpa veniam porro iste vel"
    },
    {
        "postId": 100,
        "id": 497,
        "name": "doloribus dolores ut dolores occaecati",
        "email": "Dolly@mandy.co.uk",
        "body": "non dolor consequatur\nlaboriosam ut deserunt autem odit\nlibero dolore non nesciunt qui\naut est consequatur quo dolorem"
    },
    {
        "postId": 100,
        "id": 498,
        "name": "dolores minus aut libero",
        "email": "Davion@eldora.net",
        "body": "aliquam pariatur suscipit fugiat eos sunt\noptio voluptatem eveniet rerum dignissimos\nquia aut beatae\nmodi consequatur qui rerum sint veritatis deserunt est"
    },
    {
        "postId": 100,
        "id": 499,
        "name": "excepturi sunt cum a et rerum quo voluptatibus quia",
        "email": "Wilburn_Labadie@araceli.name",
        "body": "et necessitatibus tempora ipsum quaerat inventore est quasi quidem\nea repudiandae laborum omnis ab reprehenderit ut\nratione sit numquam culpa a rem\natque aut et"
    },
    {
        "postId": 100,
        "id": 500,
        "name": "ex eaque eum natus",
        "email": "Emma@joanny.ca",
        "body": "perspiciatis quis doloremque\nveniam nisi eos velit sed\nid totam inventore voluptatem laborum et eveniet\naut aut aut maxime quia temporibus ut omnis"
    }
]
```





（2）下面根据帖子 ID 获取指定贴子的评论（有两种方式），里面包含贴子 ID、评论 ID、评论人姓名、评论人邮箱、评论内容。

1. http://jsonplaceholder.typicode.com/posts/4/comments
2. http://jsonplaceholder.typicode.com/comments?postId=4

```json
[
    {
        "postId": 4,
        "id": 16,
        "name": "perferendis temporibus delectus optio ea eum ratione dolorum",
        "email": "Christine@ayana.info",
        "body": "iste ut laborum aliquid velit facere itaque\nquo ut soluta dicta voluptate\nerror tempore aut et\nsequi reiciendis dignissimos expedita consequuntur libero sed fugiat facilis"
    },
    {
        "postId": 4,
        "id": 17,
        "name": "eos est animi quis",
        "email": "Preston_Hudson@blaise.tv",
        "body": "consequatur necessitatibus totam sed sit dolorum\nrecusandae quae odio excepturi voluptatum harum voluptas\nquisquam sit ad eveniet delectus\ndoloribus odio qui non labore"
    },
    {
        "postId": 4,
        "id": 18,
        "name": "aut et tenetur ducimus illum aut nulla ab",
        "email": "Vincenza_Klocko@albertha.name",
        "body": "veritatis voluptates necessitatibus maiores corrupti\nneque et exercitationem amet sit et\nullam velit sit magnam laborum\nmagni ut molestias"
    },
    {
        "postId": 4,
        "id": 19,
        "name": "sed impedit rerum quia et et inventore unde officiis",
        "email": "Madelynn.Gorczany@darion.biz",
        "body": "doloribus est illo sed minima aperiam\nut dignissimos accusantium tempore atque et aut molestiae\nmagni ut accusamus voluptatem quos ut voluptates\nquisquam porro sed architecto ut"
    },
    {
        "postId": 4,
        "id": 20,
        "name": "molestias expedita iste aliquid voluptates",
        "email": "Mariana_Orn@preston.org",
        "body": "qui harum consequatur fugiat\net eligendi perferendis at molestiae commodi ducimus\ndoloremque asperiores numquam qui\nut sit dignissimos reprehenderit tempore"
    }
]
```



## 3、获取专辑数据

（1）下面获取所有专辑数据，共 100 条。专辑数据与贴子数据相比就是少了内容项（body）http://jsonplaceholder.typicode.com/albums

```json
[
    {
        "userId": 1,
        "id": 1,
        "title": "quidem molestiae enim"
    },
    {
        "userId": 1,
        "id": 2,
        "title": "sunt qui excepturi placeat culpa"
    },
    {
        "userId": 1,
        "id": 3,
        "title": "omnis laborum odio"
    },
    {
        "userId": 1,
        "id": 4,
        "title": "non esse culpa molestiae omnis sed optio"
    },
    {
        "userId": 1,
        "id": 5,
        "title": "eaque aut omnis a"
    },
    {
        "userId": 1,
        "id": 6,
        "title": "natus impedit quibusdam illo est"
    },
    {
        "userId": 1,
        "id": 7,
        "title": "quibusdam autem aliquid et et quia"
    },
    {
        "userId": 1,
        "id": 8,
        "title": "qui fuga est a eum"
    },
    {
        "userId": 1,
        "id": 9,
        "title": "saepe unde necessitatibus rem"
    },
    {
        "userId": 1,
        "id": 10,
        "title": "distinctio laborum qui"
    },
    // 省略了数据
    {
        "userId": 10,
        "id": 91,
        "title": "repellendus praesentium debitis officiis"
    },
    {
        "userId": 10,
        "id": 92,
        "title": "incidunt et et eligendi assumenda soluta quia recusandae"
    },
    {
        "userId": 10,
        "id": 93,
        "title": "nisi qui dolores perspiciatis"
    },
    {
        "userId": 10,
        "id": 94,
        "title": "quisquam a dolores et earum vitae"
    },
    {
        "userId": 10,
        "id": 95,
        "title": "consectetur vel rerum qui aperiam modi eos aspernatur ipsa"
    },
    {
        "userId": 10,
        "id": 96,
        "title": "unde et ut molestiae est molestias voluptatem sint"
    },
    {
        "userId": 10,
        "id": 97,
        "title": "est quod aut"
    },
    {
        "userId": 10,
        "id": 98,
        "title": "omnis quia possimus nesciunt deleniti assumenda sed autem"
    },
    {
        "userId": 10,
        "id": 99,
        "title": "consectetur ut id impedit dolores sit ad ex aut"
    },
    {
        "userId": 10,
        "id": 100,
        "title": "enim repellat iste"
    }
]
```



（2）根据 ID 获取指定专辑数据。http://jsonplaceholder.typicode.com/albums/6

```json
{
    "userId": 1,
    "id": 6,
    "title": "natus impedit quibusdam illo est"
}
```



（3）根据用户 ID 获取该用户的所有专辑数据。http://jsonplaceholder.typicode.com/albums?userId=9

```json
[
    {
        "userId": 9,
        "id": 81,
        "title": "quibusdam sapiente et"
    },
    {
        "userId": 9,
        "id": 82,
        "title": "recusandae consequatur vel amet unde"
    },
    {
        "userId": 9,
        "id": 83,
        "title": "aperiam odio fugiat"
    },
    {
        "userId": 9,
        "id": 84,
        "title": "est et at eos expedita"
    },
    {
        "userId": 9,
        "id": 85,
        "title": "qui voluptatem consequatur aut ab quis temporibus praesentium"
    },
    {
        "userId": 9,
        "id": 86,
        "title": "eligendi mollitia alias aspernatur vel ut iusto"
    },
    {
        "userId": 9,
        "id": 87,
        "title": "aut aut architecto"
    },
    {
        "userId": 9,
        "id": 88,
        "title": "quas perspiciatis optio"
    },
    {
        "userId": 9,
        "id": 89,
        "title": "sit optio id voluptatem est eum et"
    },
    {
        "userId": 9,
        "id": 90,
        "title": "est vel dignissimos"
    }
]
```





## 4、获取待办事宜数据

（1）下面获取所有待办数据，共 200 条。待办数据特点是除了标题外，还有个布尔类型属性项（completed）表示任务是否完成。

- http://jsonplaceholder.typicode.com/todos

```json
[
    {
        "userId": 1,
        "id": 1,
        "title": "delectus aut autem",
        "completed": false
    },
    {
        "userId": 1,
        "id": 2,
        "title": "quis ut nam facilis et officia qui",
        "completed": false
    },
    {
        "userId": 1,
        "id": 3,
        "title": "fugiat veniam minus",
        "completed": false
    },
    {
        "userId": 1,
        "id": 4,
        "title": "et porro tempora",
        "completed": true
    },
    {
        "userId": 1,
        "id": 5,
        "title": "laboriosam mollitia et enim quasi adipisci quia provident illum",
        "completed": false
    },
    {
        "userId": 1,
        "id": 6,
        "title": "qui ullam ratione quibusdam voluptatem quia omnis",
        "completed": false
    },
    {
        "userId": 1,
        "id": 7,
        "title": "illo expedita consequatur quia in",
        "completed": false
    },
    {
        "userId": 1,
        "id": 8,
        "title": "quo adipisci enim quam ut ab",
        "completed": true
    },
    {
        "userId": 1,
        "id": 9,
        "title": "molestiae perspiciatis ipsa",
        "completed": false
    },
    {
        "userId": 1,
        "id": 10,
        "title": "illo est ratione doloremque quia maiores aut",
        "completed": true
    },
    {
        "userId": 1,
        "id": 11,
        "title": "vero rerum temporibus dolor",
        "completed": true
    },
    {
        "userId": 1,
        "id": 12,
        "title": "ipsa repellendus fugit nisi",
        "completed": true
    },
    {
        "userId": 1,
        "id": 13,
        "title": "et doloremque nulla",
        "completed": false
    },
    {
        "userId": 1,
        "id": 14,
        "title": "repellendus sunt dolores architecto voluptatum",
        "completed": true
    },
    {
        "userId": 1,
        "id": 15,
        "title": "ab voluptatum amet voluptas",
        "completed": true
    },
    {
        "userId": 1,
        "id": 16,
        "title": "accusamus eos facilis sint et aut voluptatem",
        "completed": true
    },
    {
        "userId": 1,
        "id": 17,
        "title": "quo laboriosam deleniti aut qui",
        "completed": true
    },
    {
        "userId": 1,
        "id": 18,
        "title": "dolorum est consequatur ea mollitia in culpa",
        "completed": false
    },
    {
        "userId": 1,
        "id": 19,
        "title": "molestiae ipsa aut voluptatibus pariatur dolor nihil",
        "completed": true
    },
    {
        "userId": 1,
        "id": 20,
        "title": "ullam nobis libero sapiente ad optio sint",
        "completed": true
    },
    // 省略了数据
    {
        "userId": 10,
        "id": 190,
        "title": "accusamus sint iusto et voluptatem exercitationem",
        "completed": true
    },
    {
        "userId": 10,
        "id": 191,
        "title": "temporibus atque distinctio omnis eius impedit tempore molestias pariatur",
        "completed": true
    },
    {
        "userId": 10,
        "id": 192,
        "title": "ut quas possimus exercitationem sint voluptates",
        "completed": false
    },
    {
        "userId": 10,
        "id": 193,
        "title": "rerum debitis voluptatem qui eveniet tempora distinctio a",
        "completed": true
    },
    {
        "userId": 10,
        "id": 194,
        "title": "sed ut vero sit molestiae",
        "completed": false
    },
    {
        "userId": 10,
        "id": 195,
        "title": "rerum ex veniam mollitia voluptatibus pariatur",
        "completed": true
    },
    {
        "userId": 10,
        "id": 196,
        "title": "consequuntur aut ut fugit similique",
        "completed": true
    },
    {
        "userId": 10,
        "id": 197,
        "title": "dignissimos quo nobis earum saepe",
        "completed": true
    },
    {
        "userId": 10,
        "id": 198,
        "title": "quis eius est sint explicabo",
        "completed": true
    },
    {
        "userId": 10,
        "id": 199,
        "title": "numquam repellendus a magnam",
        "completed": true
    },
    {
        "userId": 10,
        "id": 200,
        "title": "ipsam aperiam voluptates qui",
        "completed": false
    }
]
```



（2）根据 ID 获取指定待办数据。http://jsonplaceholder.typicode.com/todos/6

```json
{
    "userId": 1,
    "id": 6,
    "title": "qui ullam ratione quibusdam voluptatem quia omnis",
    "completed": false
}
```



（3）根据用户 ID 获取该用户的所有待办数据。20 条数据。http://jsonplaceholder.typicode.com/todos?userId=9

```json
[
    {
        "userId": 9,
        "id": 161,
        "title": "ex hic consequuntur earum omnis alias ut occaecati culpa",
        "completed": true
    },
    {
        "userId": 9,
        "id": 162,
        "title": "omnis laboriosam molestias animi sunt dolore",
        "completed": true
    },
    {
        "userId": 9,
        "id": 163,
        "title": "natus corrupti maxime laudantium et voluptatem laboriosam odit",
        "completed": false
    },
    {
        "userId": 9,
        "id": 164,
        "title": "reprehenderit quos aut aut consequatur est sed",
        "completed": false
    },
    {
        "userId": 9,
        "id": 165,
        "title": "fugiat perferendis sed aut quidem",
        "completed": false
    },
    {
        "userId": 9,
        "id": 166,
        "title": "quos quo possimus suscipit minima ut",
        "completed": false
    },
    {
        "userId": 9,
        "id": 167,
        "title": "et quis minus quo a asperiores molestiae",
        "completed": false
    },
    {
        "userId": 9,
        "id": 168,
        "title": "recusandae quia qui sunt libero",
        "completed": false
    },
    {
        "userId": 9,
        "id": 169,
        "title": "ea odio perferendis officiis",
        "completed": true
    },
    {
        "userId": 9,
        "id": 170,
        "title": "quisquam aliquam quia doloribus aut",
        "completed": false
    },
    {
        "userId": 9,
        "id": 171,
        "title": "fugiat aut voluptatibus corrupti deleniti velit iste odio",
        "completed": true
    },
    {
        "userId": 9,
        "id": 172,
        "title": "et provident amet rerum consectetur et voluptatum",
        "completed": false
    },
    {
        "userId": 9,
        "id": 173,
        "title": "harum ad aperiam quis",
        "completed": false
    },
    {
        "userId": 9,
        "id": 174,
        "title": "similique aut quo",
        "completed": false
    },
    {
        "userId": 9,
        "id": 175,
        "title": "laudantium eius officia perferendis provident perspiciatis asperiores",
        "completed": true
    },
    {
        "userId": 9,
        "id": 176,
        "title": "magni soluta corrupti ut maiores rem quidem",
        "completed": false
    },
    {
        "userId": 9,
        "id": 177,
        "title": "et placeat temporibus voluptas est tempora quos quibusdam",
        "completed": false
    },
    {
        "userId": 9,
        "id": 178,
        "title": "nesciunt itaque commodi tempore",
        "completed": true
    },
    {
        "userId": 9,
        "id": 179,
        "title": "omnis consequuntur cupiditate impedit itaque ipsam quo",
        "completed": true
    },
    {
        "userId": 9,
        "id": 180,
        "title": "debitis nisi et dolorem repellat et",
        "completed": true
    }
]
```





## 5、获取用户数据

（1）下面获取所有用户数据，共 10 条。数据包含姓名、电话、地址等各种用户信息。

- http://jsonplaceholder.typicode.com/users

```json
[
    {
        "id": 1,
        "name": "Leanne Graham",
        "username": "Bret",
        "email": "Sincere@april.biz",
        "address": {
            "street": "Kulas Light",
            "suite": "Apt. 556",
            "city": "Gwenborough",
            "zipcode": "92998-3874",
            "geo": {
                "lat": "-37.3159",
                "lng": "81.1496"
            }
        },
        "phone": "1-770-736-8031 x56442",
        "website": "hildegard.org",
        "company": {
            "name": "Romaguera-Crona",
            "catchPhrase": "Multi-layered client-server neural-net",
            "bs": "harness real-time e-markets"
        }
    },
    {
        "id": 2,
        "name": "Ervin Howell",
        "username": "Antonette",
        "email": "Shanna@melissa.tv",
        "address": {
            "street": "Victor Plains",
            "suite": "Suite 879",
            "city": "Wisokyburgh",
            "zipcode": "90566-7771",
            "geo": {
                "lat": "-43.9509",
                "lng": "-34.4618"
            }
        },
        "phone": "010-692-6593 x09125",
        "website": "anastasia.net",
        "company": {
            "name": "Deckow-Crist",
            "catchPhrase": "Proactive didactic contingency",
            "bs": "synergize scalable supply-chains"
        }
    },
    {
        "id": 3,
        "name": "Clementine Bauch",
        "username": "Samantha",
        "email": "Nathan@yesenia.net",
        "address": {
            "street": "Douglas Extension",
            "suite": "Suite 847",
            "city": "McKenziehaven",
            "zipcode": "59590-4157",
            "geo": {
                "lat": "-68.6102",
                "lng": "-47.0653"
            }
        },
        "phone": "1-463-123-4447",
        "website": "ramiro.info",
        "company": {
            "name": "Romaguera-Jacobson",
            "catchPhrase": "Face to face bifurcated interface",
            "bs": "e-enable strategic applications"
        }
    },
    {
        "id": 4,
        "name": "Patricia Lebsack",
        "username": "Karianne",
        "email": "Julianne.OConner@kory.org",
        "address": {
            "street": "Hoeger Mall",
            "suite": "Apt. 692",
            "city": "South Elvis",
            "zipcode": "53919-4257",
            "geo": {
                "lat": "29.4572",
                "lng": "-164.2990"
            }
        },
        "phone": "493-170-9623 x156",
        "website": "kale.biz",
        "company": {
            "name": "Robel-Corkery",
            "catchPhrase": "Multi-tiered zero tolerance productivity",
            "bs": "transition cutting-edge web services"
        }
    },
    {
        "id": 5,
        "name": "Chelsey Dietrich",
        "username": "Kamren",
        "email": "Lucio_Hettinger@annie.ca",
        "address": {
            "street": "Skiles Walks",
            "suite": "Suite 351",
            "city": "Roscoeview",
            "zipcode": "33263",
            "geo": {
                "lat": "-31.8129",
                "lng": "62.5342"
            }
        },
        "phone": "(254)954-1289",
        "website": "demarco.info",
        "company": {
            "name": "Keebler LLC",
            "catchPhrase": "User-centric fault-tolerant solution",
            "bs": "revolutionize end-to-end systems"
        }
    },
    {
        "id": 6,
        "name": "Mrs. Dennis Schulist",
        "username": "Leopoldo_Corkery",
        "email": "Karley_Dach@jasper.info",
        "address": {
            "street": "Norberto Crossing",
            "suite": "Apt. 950",
            "city": "South Christy",
            "zipcode": "23505-1337",
            "geo": {
                "lat": "-71.4197",
                "lng": "71.7478"
            }
        },
        "phone": "1-477-935-8478 x6430",
        "website": "ola.org",
        "company": {
            "name": "Considine-Lockman",
            "catchPhrase": "Synchronised bottom-line interface",
            "bs": "e-enable innovative applications"
        }
    },
    {
        "id": 7,
        "name": "Kurtis Weissnat",
        "username": "Elwyn.Skiles",
        "email": "Telly.Hoeger@billy.biz",
        "address": {
            "street": "Rex Trail",
            "suite": "Suite 280",
            "city": "Howemouth",
            "zipcode": "58804-1099",
            "geo": {
                "lat": "24.8918",
                "lng": "21.8984"
            }
        },
        "phone": "210.067.6132",
        "website": "elvis.io",
        "company": {
            "name": "Johns Group",
            "catchPhrase": "Configurable multimedia task-force",
            "bs": "generate enterprise e-tailers"
        }
    },
    {
        "id": 8,
        "name": "Nicholas Runolfsdottir V",
        "username": "Maxime_Nienow",
        "email": "Sherwood@rosamond.me",
        "address": {
            "street": "Ellsworth Summit",
            "suite": "Suite 729",
            "city": "Aliyaview",
            "zipcode": "45169",
            "geo": {
                "lat": "-14.3990",
                "lng": "-120.7677"
            }
        },
        "phone": "586.493.6943 x140",
        "website": "jacynthe.com",
        "company": {
            "name": "Abernathy Group",
            "catchPhrase": "Implemented secondary concept",
            "bs": "e-enable extensible e-tailers"
        }
    },
    {
        "id": 9,
        "name": "Glenna Reichert",
        "username": "Delphine",
        "email": "Chaim_McDermott@dana.io",
        "address": {
            "street": "Dayna Park",
            "suite": "Suite 449",
            "city": "Bartholomebury",
            "zipcode": "76495-3109",
            "geo": {
                "lat": "24.6463",
                "lng": "-168.8889"
            }
        },
        "phone": "(775)976-6794 x41206",
        "website": "conrad.com",
        "company": {
            "name": "Yost and Sons",
            "catchPhrase": "Switchable contextually-based project",
            "bs": "aggregate real-time technologies"
        }
    },
    {
        "id": 10,
        "name": "Clementina DuBuque",
        "username": "Moriah.Stanton",
        "email": "Rey.Padberg@karina.biz",
        "address": {
            "street": "Kattie Turnpike",
            "suite": "Suite 198",
            "city": "Lebsackbury",
            "zipcode": "31428-2261",
            "geo": {
                "lat": "-38.2386",
                "lng": "57.2232"
            }
        },
        "phone": "024-648-3804",
        "website": "ambrose.net",
        "company": {
            "name": "Hoeger LLC",
            "catchPhrase": "Centralized empowering task-force",
            "bs": "target end-to-end models"
        }
    }
]
```





（2）根据 ID 获取指定用户数据。

- http://jsonplaceholder.typicode.com/users/5

```json
{
    "id": 5,
    "name": "Chelsey Dietrich",
    "username": "Kamren",
    "email": "Lucio_Hettinger@annie.ca",
    "address": {
        "street": "Skiles Walks",
        "suite": "Suite 351",
        "city": "Roscoeview",
        "zipcode": "33263",
        "geo": {
            "lat": "-31.8129",
            "lng": "62.5342"
        }
    },
    "phone": "(254)954-1289",
    "website": "demarco.info",
    "company": {
        "name": "Keebler LLC",
        "catchPhrase": "User-centric fault-tolerant solution",
        "bs": "revolutionize end-to-end systems"
    }
}
```





## 6、获取照片数据

（1）下面获取所有相簿里的所有照片数据，共 5000 条。数据包含照片 ID、相簿 ID、标题、照片原图地址、照片缩略图地址。

- http://jsonplaceholder.typicode.com/photos

```json
[
    {
        "albumId": 1,
        "id": 1,
        "title": "accusamus beatae ad facilis cum similique qui sunt",
        "url": "https://via.placeholder.com/600/92c952",
        "thumbnailUrl": "https://via.placeholder.com/150/92c952"
    },
    //...中间省略了数据
    {
        "albumId": 100,
        "id": 5000,
        "title": "error quasi sunt cupiditate voluptate ea odit beatae",
        "url": "https://via.placeholder.com/600/6dd9cb",
        "thumbnailUrl": "https://via.placeholder.com/150/6dd9cb"
    }
]
```

PS注意：数据里面的两个 url 也是可以访问的，比如缩略图地址访问结果如下。

![img](https://via.placeholder.com/150/6dd9cb)

（2）根据 ID 获取指定照片数据。http://jsonplaceholder.typicode.com/photos/8

```json
{
    "albumId": 1,
    "id": 8,
    "title": "aut porro officiis laborum odit ea laudantium corporis",
    "url": "https://via.placeholder.com/600/54176f",
    "thumbnailUrl": "https://via.placeholder.com/150/54176f"
}
```





（3）根据相簿 ID 获取该相簿下所有的照片数据。

- http://jsonplaceholder.typicode.com/photos?albumId=5

```json
[
    {
        "albumId": 5,
        "id": 201,
        "title": "nesciunt dolorum consequatur ullam tempore accusamus debitis sit",
        "url": "https://via.placeholder.com/600/250289",
        "thumbnailUrl": "https://via.placeholder.com/150/250289"
    },
    // ...中间省略了数据
    {
        "albumId": 5,
        "id": 250,
        "title": "voluptatem repellendus voluptatibus id occaecati ipsam dignissimos officia",
        "url": "https://via.placeholder.com/600/e33ffb",
        "thumbnailUrl": "https://via.placeholder.com/150/e33ffb"
    }
]
```



# 三、POST 请求数据

我们可以将数据通过 POST 方式提交到如下地址，它除了会把我们提交的数据返回来，还会在结果中附带个随机的 id 值。

1. 添加帖子接口请求地址：https://jsonplaceholder.typicode.com/posts

```bash
$ curl https://jsonplaceholder.typicode.com/posts -H "Content-Type: application/json" -X POST -d '{"name": "Sam", "age":18, "msg":"OK" }'
{
  "name": "Sam",
  "age": 18,
  "msg": "OK",
  "id": 101
}
```

请求的request body为：

```json
{
    "name": "Sam", 
    "age":18, 
    "msg":"OK" 
}
```

返回的response body为：

```json
{
    "name": "Sam",
    "age": 18,
    "msg": "OK",
    "id": 101
}
```



# 四、其他免费 API 接口

> 分享免费api接口：https://mp.weixin.qq.com/s/FEGZm6YJDIOF0cYDZUrVrg

一、UomgAPI【稳定、快速、免费的 API 接口服务】：https://api.uomg.com/

二、极速数据：https://www.jisuapi.com/api/?isfree=1

三、小白API接口【稳定、快速、免费的 API 接口服务共收录了 32 个接口项目】：https://api.xiaobaibk.com/

四、黑马优购：https://www.showdoc.com.cn/128719739414963/2513235043485226

五、网易云音乐API：https://binaryify.github.io/NeteaseCloudMusicApi/#/

六、其他免费接口API：

1、猫眼电影接口：

- 正在热映：https://m.maoyan.com/ajax/movieOnInfoLis
- 影院：https://m.maoyan.com/ajax/filterCinemas?ci=57
  - 参数：ci ⇒ 城市id
- 影院详情：https://m.maoyan.com/ajax/cinemaDetail?cinemaId=890
  - 参数：cinemaId ⇒ 影院id
- 即将上映：https://m.maoyan.com/ajax/comingList?ci=10&token=&limit=10
  - 参数：ci ⇒ 城市id、limit ⇒ 请求几条
- 电影详情：https://m.maoyan.com/ajax/detailmovie?movieId=1203734

2、B站视频接口：

- 推荐视频：https://www.bilibili.com/index/ding.json
- 默认搜索词：http://api.bilibili.cn/x/web-interface/search/default
- 搜索热词：https://s.search.bilibili.com/main/hotword
- 分区视频排行榜：https://api.bilibili.com/x/web-interface/ranking

3、网易新闻接口：

- 头条：http://c.m.163.com/nc/article/headline/T1348647853363/0-40.html
- 精选：http://c.3g.163.com/nc/article/list/T1467284926140/0-20.html
- 娱乐：http://c.3g.163.com/nc/article/list/T1348648517839/0-20.html
- 汽车：http://c.m.163.com/nc/auto/list/5bmz6aG25bGx/0-20.html
- 新闻：http://c.m.163.com/nc/auto/list/6YOR5bee/0-20.html
- 新闻：http://c.m.163.com/nc/auto/list/6YOR5bee/20-20.html
- 运动：http://c.3g.163.com/nc/article/list/T1348649079062/0-20.html
- 视频：http://c.m.163.com/recommend/getChanListNews?channel=T1457068979049&size=20

3、360图片接口：

- 获取某类壁纸：http://wallpaper.apc.360.cn/index.php?c=WallPaperAndroid&a=getAppsByCategory&cid=9&start=0&count=99
  - total：返回数据数量
  - data：返回的数据
  - pid：
  - cid：类别ID
  - url：壁纸地址
  - fav_total：收藏数
- 搜索壁纸：http://wallpaper.apc.360.cn/index.php?c=WallPaper&a=search&start=0&count=99&kw=by&start=0&count=99
  - kw：关键字
  - start：跳过的记录数
  - count：返回的数量
- 获取今日搜索热词：http://openbox.mobilem.360.cn/html/api/wallpaperhot.html
