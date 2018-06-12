# 20180612 with LikeLion day5

## 오늘 할 내용

* CRUD 중에서 CR을 합니다.
* 자료가 저장되는 곳은 DB가 아니라 CSV파일로 저장합니다.
* 그리고 사용자의 입력을 받아서 간이 게시판을 만듭니다.


	# *CSV에서 파일을 읽어올 때의 옵션*	

* r

Read-only mode. The file pointer is placed at the beginning of the file. This is the default mode.

* r+

Read-write mode. The file pointer will be at the beginning of the file.

* w

Write-only mode. Overwrites the file if the file exists. If the file does not exist, creates a new file for writing.

* w+

Read-write mode. **Overwrites the existing file if the file exists. If the file does not exist, creates a new file for reading and writing.**

* a

Write-only mode. The file pointer is at the end of the file if the file exists. That is, the file is in the append mode. If the file does not exist, it creates a new file for writing.

* a+

Read and write mode. **The file pointer is at the end of the file if the file exists. The file opens in the append mode. If the file does not exist, it creates a new file for reading and writing.**

## 1. 게시판의 역할 정의하기 및 bootstrap 활용하기

- 게시판은 단순히 제목과, 본문의 속성을 가진, CRUD를 필요로 하는 가장 기본적인 기능이다.
- CRUD란 `Create`, `Read`, `Update`, `Delete` 의 역할을 의미한다.
- 제목과 본문을 DB에 저장하고 필요에 따라 해당 부분을 불러오게 된다.
- 게시판 기능을 만들기 이전에 게시판 글 하나가 가져야할 속성을 정리해보자.

> 인덱싱을 위한 ID, 제목, 본문, 작성자 등이 있을 텐데 우리는 ID, 제목, 본문만 저장하고 구현해보자.
>
> 또한 조금 더 아름다운 view를 만들기 위해 bootstrap을 사용할 것이다.

- 코드에서 가장 중요한 부분중 하나는 반복되는 코드를 최소한으로 줄이는 것이다. 우리는 `<html>`, `<head>`, `<body>` 태그가 반복적으로 쓰이는 것을 막기위해 `layout`, 즉 틀을 지정할 것이다. 

*terminal*

```
command
$ mkdir day4
$ mkdir day4/views
$ cd day4
$ touch app.rb && touch views/layout.erb
$ gem install 'sinatra'
$ gem install 'sinatra-reloader'
```

*views/layout.erb*

```html
<html>
    <head>
        <title>로보어드바이저 과정 화이팅</title>
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css" integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
        <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js" integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js" integrity="sha384-smHYKdLADwkXOn1EmN1qk/HfnUcbVRZyYmZ4qpPea6sjB/pTJ0euyQp0Mk8ck+5T" crossorigin="anonymous"></script>
    </head>
    <body>
        <%= yield %>
    </body>
</html>
```

*views/app.erb*

```html
<p>Hi!</p>
```

*app.rb*

```ruby
require 'sinatra'
require 'sinatra/reloader'

get '/' do
     erb :app
end
```

- `<%= yield %>` 라는 문법이 생소할 수 있다. 하지만 말그대로 '양보한다'는 의미로 코드의 진행과정을 양보한다라고 볼 수 있겠다. 우리가 반복을 원하는 코드 중에서 변경을 원하는 부분을 찾아 해당 위치를 지정해준다고 할 수 있겠다.
- `layout.erb` 파일을 활용하는 이유는 bootstrap을 활용하기 위해서 CDN코드를 활용해야 하는데 layout을 활용하지 않으면 이 코드가 계속해서 반복되게 된다. 이러한 반복을 막기위해 layout이라고 하는 '틀'을 활용하게 되는 것이다.



## 2. CSV를 통해 DB구조에 대해 살펴보자

- DB를 사용하는 가장 근본적인 이유는 정보를 **영구적으로** 저장하기 위해서 이다. 만약 서버가 다운된다고 해서 작성했던 모든 정보가 삭제된다면, 회원정보, 결제정보 등 많은 중요한 정보를 매 순간 날려먹을 수도 있다는 불안함 속에 살아야 할 것이다.
- CSV파일을 활용하는 이유는 DB와 가장 유사하다고 할 수 있기 때문이다. 우리가 가장 가깝게, 자주 활용하던 DBMS가 바로 excel이다. 컬럼에 속성을 지정하고 그 속성에 맞춰 1줄씩 데이터를 추가하고, 원하는 데이터를 조회하고, 수정, 삭제하는 것이 바로 DBMS가 하는 역할이다.
- CSV를 활용하기 위해서 `gem install csv` 를 먼저 진행한다.

```ruby
require 'sinatra'
require 'sinatra/reloader'
require 'csv'
...
```

- 이제 우리의 프로젝트에서 CSV 라이브러리를 활용할 수 있다.

> 본격적으로 Create와. Read에 대해서 배우기 이전에 WildCard Parameter에 대해서 알고가자.
>
> 활용 형태는 다음과 같다.
>
> ```ruby
> get '/index/:id' do
> params[:id]
>     ...
> end
> ```
>
> 기존에 사용해왔던 Parameter와는 다르게 url자체에 파라미터를 받을 수 있다. 우리가 활용하는 Github도 내 계정에 접속했을 때 처음 접근하는 url이 `https://github.com/lovings2u`로 구성되어 있는데 여기서 우리의 계정명이 하나의 파라미터로 활용되는 것이라고 볼 수 있겠다. 이는 [일부 블로그 사이트](https://medium.com/s/story/why-breakups-hurt-like-hell-5edcd3777a46) 등에서도 발견할 수 있다. 물론 동일한 제목일 경우 중첩이 될 수 있기 때문에 이를 구분할 id를 추가하는 것이 중요하다.

- CSV는 다음과 같은 형태로 되어 있다.

```csv
1,title1,contents1
2,title2,contents2
3,title3,contents3
4,title4,contents4
...
```

- 각 한줄은 row라고 하고 하나의 ,(comma)는 하나의 컬럼을 의미한다. 한 줄은 하나의 데이터를 의미하고 하나의 세로선은 하나의 속성을 의미한다.
- 오늘 배울 내용은 `Create` 와 `Read`에 대해서 배울 건데 각각의 액션을 먼저 정의 해보자.
  - 모든 글을 보여줄 인덱스 페이지는 `/index`
  - 하나의 글을 보여줄 페이지는 `/show/글번호`
  - 새 글을 등록할 페이지는 `/new`
  - 실제로 글을 등록할 곳은 `/create`

## 3. Index 와 Show

- 아직 우리가 직접 데이터를 등록하는 것은 불가능 하지만 직접 CSV파일을 작성하는 것은 가능하다. 일단 직접 CSV파일을 작성하고 그 이후에 Create에 대해 배워보자
- CSV 라이브러리가 가지고 있는 메소드에 대해서는 [다음](https://ruby-doc.org/stdlib-2.0.0/libdoc/csv/rdoc/CSV.html) 에서 확인할 수 있다.

*views/index.erb*

```html
<table class="table">
    <thead>
        <th>글번호</th>
        <th>제목</th>
        <th></th>
    </thead>
    <tbody>
        <% @boards.each do |board| %>
        <tr>
            <td><%= board[0] %></td>
            <td><%= board[1] %></td>
            <td><a href="/boards/<%= board[0] %>">글보기</a></td>
        </tr>
        <% end %>
    </tbody>
</table>
```

*app.rb*

```ruby
...
get '/index' do
    @boards = []
    CSV.foreach('app.csv') do |row|
        @boards << row
    end
    erb :index
end
...
```

*app.csv*

```csv
1,title1,contents1
2,title2,contents2
3,title3,contents3
4,title4,contents4
​```

- CSV에 등록되어 있는 모든 글을 `@boards`라고 하는 배열에 저장하고 배열의 각 요소를 순회하며 글번호, 제목, 본문을 출력한다.
- 하나의 글을 조회할 때에는 `/show/글번호` 형태의 url을 사용한다. 이를 위해서는 위에서 간략히 살펴봤던 WildCard를 활용하도록 한다.

*views/show.erb*

​```html
<h1>제목: <%= @board[1] %></h1>
<p>본문: <%= @board[2] %></p>
```

*app.rb*

```ruby
get '/boards/:id' do
    @board = []
    CSV.foreach('app.csv') do |row|
        @board = row if row[0].eql?(params[:id])
    end
    erb :show
end
```

- DBMS에서 하나의 글을 조회할 때에는 기본적으로 Btree Search를 이용한다. 하지만 우리는 적은 개수의 데이터만 가지고 테스트 할 예정이기 때문에 전체조회를 활용한다.
- 전체를 조회하다가 사용자가 원하는 글번호에 해당하는 row를 만나면 해당 줄을 `@board` 변수에 값을 저장한다.
- 저장한 값을 `show.erb` 에서 보여준다.



## 4. New와 Create

- 새 글을 등록할 때에는 먼저 사용자로부터 내용을 받을 페이지가 필요하다. 그리고 사용자가 입력한 정보를 바탕으로 새로운 액션에서 저장하고 `show` 페이지나 `index` 페이지로 리디렉션을 발생시킨다.

*views/new*

```html
<form action="/create" method="POST">
    <input type="text" name="title" placeholder="제목"><br>
    <textarea name="contents" row="4" column="10" placeholder="본문"></textarea>
    <input type="submit" value="작성">
</form>
```

*app.rb*

```ruby
get '/new' do
    erb :new
end

post '/create' do
    index = CSV.read('app.csv').size
    CSV.open('app.csv', 'a+') do |row|
        row << [index+1,params[:title], params[:contents]]
    end
    redirect "/index"
end
```

- 새로운 정보를 등록하는 페이지에는 기타 로직이 필요하지 않다. 현재는 그러하다.

- 사용자는 작성이 완료되어 form을 submit하면 `/create` 액션에 `POST` method로 요청을 보내게 된다. 

- 새 글을 등록할 때에는 해당 글이 몇번째 글로 등록될 것인지 확인하기 위해 현재까지 등록된 글의 개수를 파악하고 그 숫자에 1을 더해 새 글을 등록하게 된다.


  ## create 와 Read

  1. `get` /boards/new
  2. `get` /boards/create
  3. `get`/boards
  4. `get` /boards/:id

  * `board`라고 하는 게시판이 하나만 존재하고 있다.
  * `user`라고 하는 `CRUD` 기능을 해야하는 DB Table을 만든다고 가정하면?
  * 새로운 유저를 등록한다면?

## 스스로 과제

* `bootstrap`을 사용할 수 있도록 `CDN`으로 추가하기.

* 사용자의 입력을 받는 `form`태그로 이루어진 `/new` 액션과 `erb`파일
  * `form`의 `action`속성은 `/create` 로 가도록 합니다.
  * `method`는  `post`를 이용합니다.
  * 게시판 글의 제목(`title`)과 본문(`contents`) 두가지 속성을 저장합니다.
* 전체목록을 보여주는 `table`태그로 이루어진 `/boards`액션과 `erb`파일
* `/create` 액션을 만들고 작성 후에는 `/boards` 액션으로 돌아가게 구성
  * (수정) `/create`액션이 동작한 이후에는 본인이 작성한 글로 이동합니다.
* 각 글( 1개의 글을 보는 페이지)를 볼 수 있는 페이지는 `/board/글번호`의 형태로 작성을 합니다.

  *board.erb*

  ```html
<h1>제목: <%= @board[1]%></h1>
<p>본문: <%= @board[2]%></p>
  ```


*boards.erb*

```html
<table>
    <thead>
        <th>글번호</th>
        <th>제목</th>
        <th>보러가기</th>
    </thead>
    <tbody>
        <% @boards.each do |board| %>
        <tr>
            <td><%= board[0]%></td>
            <td><%= board[1]%></td>
            <td></td>
        </tr>
        <% end %>
    </tbody>
</table>
```

*new.erb*

```html
<form action='/create' method='POST'>
    제목<input type ="text" name= "title"><br/>
    본문<textarea name ="contents" placeholder ="본문"></textarea>
    <input type="submit" value="등록하기">
</form>
```

 *app.rb*

```ruby
require "sinatra"
require "sinatra/reloader"
require "rest-client"
require 'json'
require 'uri'
require "csv"

get '/' do
    erb :index
end

get '/new' do
    erb :new    
end

post '/create' do
    # 사용자가 입력한 정보를 받아서
    # CSV파일 가장 마지막에 등록
    # => 이 글의 글번호도 같이 저장해야함
    # => 기존의 글 개수를 파악해서
    # => 글 개수 + 1 해서 저장
    title = params[:title]
    contents = params[:contents]
    id = CSV.read('./boards.csv').count + 1
    puts id
    CSV.open('./boards.csv', 'a+') do |row|
        row << [id, title, contents]
    end
    redirect '/boards'
end

get '/boards' do
    # 파일을 읽기 모드로 열고
    # 각 줄마다 순회하면서
    # @ 가 붙어있는 변수에 넣어준다.
    @boards = Array.new
    CSV.open('./boards.csv', 'r+').each do |row|
        @boards << row
    end
 erb :boards
end

get '/board:id' do
    # CSV파일에서 params[:id]로 넘어온 친구와 같은 글번호를 가진 row를 선택
    # => CSV파일을 전체 순회합니다.
    # => 순회하다가 첫번째 column이 id와 같은 값을 만나면, 순회를 정지하고
    # => 값을 변수에다가 담아줍니다.
    @board = Array.new
    CSV.read('./boards.csv').each do |row|
        #break if row[0] == params[:id] # 멈추는 건 똑같이 break
        if row[0].eql?(params[:id])
            @board = row
            break
        end
    end
    erb :board
end
```

## User를 등록할 수 있는 csv file을 만들어 보자

* `user`를 등록할 수 있는 CSV 파일을 만들어보자

* id, password, password_confirmation

* 조건1 

  * password와 password_conformation을 받는데, 회원을 등록할 때 이 두 분자열이 다르면 회원등록이 안된다. 

* Route(라우팅)

  * `get`  /user/new -> new_user.erb
  * `post `/user/create
  * `get` /users -> users.erb
  * `get` /users/:id - > user.erb

  *app.rb*

```ruby
require "sinatra"
require "sinatra/reloader"
require "rest-client"
require 'json'
require 'uri'
require "csv"


get '/user/new' do
    erb :new_user
end

get '/error' do
    erb:error
end

post '/user/create' do
   
    id = params[:id]
    password = params[:password]
    password_confirm = params[:password_confirm]
   
    # id_password.csv가 존재하냐?
    # 안하면 만들어준다.
    unless File.exist?('id_password.csv')
        if password.eql?password_confirm
            CSV.open("id_password.csv", "a+") do |row|
                row << [id, password] 
            end
            redirect "/users/#{id}"
        else
            @msg = "비밀번호와 확인용 비밀번호가 일치하지 않습니다."
            erb:error
        end
    else
    # 존재하면 id의 존재여부를 확인하고 다시 회원가입으로
        # 읽어서 배열에 회원들의 정보를 넣어준다.
        ids = Array.new
        CSV.open('./boards.csv', 'r+').each do |row|
            ids << row
        end
        
        # 입력받은 id가 ids에 존재하지 않는다면 가입시킨다.
        unless ids[0].include?id
            if password.eql?password_confirm
                CSV.open("id_password.csv", "a+") do |row|
                    row << [id, password] 
                end
                redirect "/users/#{id}"
            else
                @msg = "비밀번호와 확인용 비밀번호가 일치하지 않습니다."
                erb:error
            end
        else
            @msg = "이미 있는 아이디입니다. 다른 아이디를 만들어주세요"
            erb :error
        end
    end
end

get '/users' do
    @ids = Array.new
    CSV.open("id_password.csv",'r+').each do |row|
        @ids << row
    end
    erb :users
end

get '/users/:id' do
   @ids = Array.new
   CSV.read('id_password.csv').each do |row|
        if row[0].eql?(params[:id])
           @ids = row
        break
        else
            @msg = "없는 아이디 입니다."
            erb:error
        end
    end
    erb :user
end
```

*layout.erb*

```html
<html>
    <head>
        <title>CSV게시판</title>
        <!-- CSS -->
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css" integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
        <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js" integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js" integrity="sha384-smHYKdLADwkXOn1EmN1qk/HfnUcbVRZyYmZ4qpPea6sjB/pTJ0euyQp0Mk8ck+5T" crossorigin="anonymous"></script>
    </head>
    <body>
        <%=yield%>
    </body>
</html>
```



*new_user.erb*

```html
<form action='/user/create' method='POST'>
    id : <input type ="text" name= "id">
    password : <input type ="password" name="password">
    password_confirm : <input type='password' name='password_confirm'>
    <input type="submit" value="회원등록">
</form>
```

*user.erb*

```html
<h1>회원가입을 축하드립니다!</h1>
<p>아이디: <%=@ids[0]%></p>
<p>비밀번호: <%= @ids[1]%></p>
```

*users.erb*

```html
<table class = "table table-hover">
    <thead>
        <th>id</th>
        <th>password</th>
    </thead>
    <tbody>
        <% @ids.each do |info| %>
        <tr>
            <td><%= info[0]%></td>
            <td><%= info[1]%></td>
        </tr>
        <% end %>
    </tbody>
</table>	
```

*error.erb*

```html
<h1><%=@msg%></h1>
```



## Ruby On Rails

1. Gem Install Bundler 

* `bundler`는 내 프로젝트에 사용될 모든 `gem`들을 자동으로 설치해줍니다.

* 내 프로젝트에 사용될 모든`gem`은 `gemfile`에 명시한다.

* `Gemlfile`에 내가 사용할 `gem`을 명시한다음에 터미널에 다음 명령어를 입력합니다.

 ```ruby
$ bundle install
 ```

* 사용할 라이브러리를 추가한 이후에도 반드시 해당 명령어를 실행해야합니다.
* 이를 통해서 설치하면 해당 라이브러리 뿐만 아니라 사용하기 위해 필요한 모든 라이브러리를 다 설치해줍니다. 
* 삭제할때는,  `gemfile`에서 지운뒤 다시 `bundle install`을 실행해줘야합니다.
2. Rails workspace의 app폴더 아래에 models, views, controller 3개가 다 들어가 있다.

  그외

  - bin  = 명령어 담당
  - config = 설정
  - log  = 서버 로그 저장
  - public = 외부에서  접근할 수 있는 폴더

3. TDD is Dead?

### Rails가 어떻게 돌아가는가?

```
View<=>Client -> Routing(Routes) - > Controller <=> Models <=> dbms
```

Route.rb = 사용자의 요청을 받아서, 어떤 컨트롤러의 어떤 액션으로 가라고 지정해준다.

### ORM?

 **1. ORM이란?** 

​	ORM(Object-relational mapping)을 단순하게 표현하면 객체와 관계와의 설정이라 할 수 있다. ORM에서 말하는 객체(Object)의 의미는 우리가 흔히 알고 있는 OOP(Object_Oriented Programming)의 그 객체를 의미한다는 것을 쉽게 유추할 수 있을 것이다. 그렇다면 과연 관계라는 것이 의미하는 것은 무엇일까? 

지극히 기초적인 이야기지만 개발자가 흔히 사용하고 있는 관계형 데이터베이스를 의미한다.

그렇다면 도대체 무엇이 문제여서 객체와 관계형 데이터베이스 간의 매핑을 지원해주는 Framework나 Tool들이 나오는 것일까? ORM framework나 도구가 없던 시절에도 이미 우리는 OOP를 하면서 객체와 관계형 데이터베이스를 모두 잘 사용하고 있었음에도 불구하고, 굳이 이런 새로운 개념들이 나오게 되는 이유는 "Back to basics(기본에 충실하자)" 을 지키기 위해서라고 볼 수 있다. 즉, 보다 OOP다운 프로그래밍을 하자는데부터 출발한 것이다.

그럼 과연 무엇이 문제였던 것일까? 우리가 어떤 어플리케이션을 만든다고 하면 관련된 정보들을 객체에 담아 보관하게 된다. 프로그래밍 실습 예제의 단골 손님격인 주소록을 만든다고 가정했을 때 주소록의 주체가 될 사람이라는 객체에는 주민등록번호, 이름, 키, 몸무게 등이 저장될 것이고 주소나 전화번호 같은 추가로 저장될 객체들이 연결 될 것이다. 이렇게 생성한 사람 객체를 영구적으로 저장하기 위해 파일이나 데이터베이스에 입력한다는 것은 객체와 그와 연결된 객체들을 데이터베이스의 테이블에 저장 한다는 것을 의미한게 된다. 

즉, 테이블(Table)에 객체가 가지고 있던 정보를 입력하고, 이 테이블들을 "join"과 같은 SQL 질의어를 통해 관계 설정을 해 주게 된다. 여기서 문제는 이 테이블과 객체간의 이질성이 발생 하게 된다는 것이다.

**2. 사용예시**

보통 ORM Framwork들은 이러한 이질성을 해결하기 위해서 객체와 테이블간의 관계를 설정하여 자동으로 처리하게 되는데 예시를 통해 확인하면 다음과 같다.

```java
	public class Person{
    private String name;
    private String height;
    private String weight;
    private String ssn;
    //implement getter & setter methods
}
```
iBatis의 경우에는 다음과 같이 mapping file내에서 해당 query의 결과를 받을 객체를 지정해 줄 수 있다
```js
<select id="getPerson" resultClass="net.agilejava.person.domain.Person">
    SELECT name, height, weight, ssn FROM USER WHERE name = #name#;
</select>
```
즉, getPerson 이라고 정의된 질의어 결과는 net.agilejava.person.domain의 Person객체에 자동으로 mapping 되는 것이다. Hibernate의 경우에는 mapping 파일에서 다음과 같이 표현을 해준다.
```javascript
<hibernate-mapping>
    <class name="net.agilejava.person.domain.Person" table="person">
        <id name="name" column="name"/>
        <property name="height" column="height"/>
        <property name="weight" column="weight"/>
        <property name="ssn" column="ssn"/>
    <class>
</hibernate-mapping>
```
위 두개의 Framework의 예시를 보면 알 수 있듯이 setter 메소드가 있으면 객체에 결과를 set하는 작업을 따로 해줄 필요 없이 자동으로 해당 값이 할당되게 된다. 물론 여기에 1:m 이나 m:1등의 관계들이 형성되면 추가적인 작업이 필요하긴 하지만 일단 눈에 보이는 간단한 부분은 처리가 되는 것을 볼 수 있다. 물론 반대의 경우에도 객체를 던져주면 ORM Framework에서 알아서 get을 수행해 해당하는 column에 넣어주게 된다.

어떻게 보면 더 복잡해 보일 수도 있는 ORM이지만 막상 사용해 보면 그 편리함에 몸을 떨게 된다. 단순하게 get/set만 해주는게 목적이 아니라 객체지향적인 시스템을 위해서 관계형 데이터베이스의 설계부터 변화를 주고, 설계된 데이터베이스와 객체와의 관계에 대한 설정 등을 포함하여 보다 객체지향적인 시스템의 완성을 위한 도구라고 말할 수 있겠다. 

물론, ORM 이라는 것이 흔히 말하는 silver bullet은 절대 아니다. 많은 사람들이 ORM에 대해 우려하고 있는 부분은 객체지향적으로 설계되지 않은 데이터베이스에서의 사용에 따른 폐해라고 생각한다. 이미 데이터베이스 중심적인 사고를 통해 만들어 놓은 데이터베이스에 ORM을 도입 해도 분명 이점이 있긴 하겠지만, 그에 비해 개발자들의 학습곡선이라던지, 기존에 존재하는 코드나 시스템들과의 연계 또는 유지보수 적인 측면, 그리고 성능 등에서 생각해보면 부정적으로 볼 수 밖에 없다. 

즉, 전체 적인 시스템의 분석, 설계 단계에서부터 객체와 데이터베이스를 따로 생각하는 것이 아니라 하나의 덩어리로 인지하고 양쪽 모두를 고려한 설계를 해나갈 수 있을 때, ORM은 보다 좋은 모습을 보여주고 각광을 받을 수 있을 것이다.

**3. 장점**
- 객체 지향적인 코드로 인해 더 직관적이고 비즈니스 로직에 더 집중할 수 있게 도와준다.
- 선언문, 할당, 종료 같은 부수적인 코드가 없거나 급격히 줄어든다.
- 각종 객체에 대한 코드를 별도로 작성하기 때문에 코드의 가독성을 올려준다.
- SQL의 절차적이고 순차적인 접근이 아닌 객체 지향적인 접근으로 인해 생산성이 증가한다.
- 재사용 및 유지보수의 편리성이 증가한다.
- ORM은 독립적으로 작성되어있고, 해당 객체들을 재활용 할 수 있다. 
- 때문에 모델에서 가공된 데이터를 컨트롤러에 의해 뷰와 합쳐지는 형태로 - 디자인 패턴을 견고하게 다지는데 유리하다.
- 매핑정보가 명확하여, ERD를 보는 것에 대한 의존도를 낮출 수 있다.
- DBMS에 대한 종속성이 줄어든다.
- 대부분 ORM 솔루션은 DB에 종속적이지 않다.
- 종속적이지 않다는것은 구현 방법 뿐만아니라 많은 솔루션에서 자료형 타입까지 유효하다.
- 프로그래머는 Object에 집중함으로 극단적으로 DBMS를 교체하는 거대한 작업에도 비교적 적은 리스크와 시간이 소요된다.
- 또한 자바에서 가공할경우 equals, hashCode의 오버라이드 같은 자바의 기능을 이용할 수 있고, 간결하고 빠른 가공이 가능하다.

**4. 단점**

- 완벽한 ORM 으로만 서비스를 구현하기가 어렵다.
- 사용하기는 편하지만 설계는 매우 신중하게 해야한다.
- 프로젝트의 복잡성이 커질경우 난이도 또한 올라갈 수 있다.
- 잘못 구현된 경우에 속도 저하 및 심각할 경우 일관성이 무너지는 문제점이 생길 수 있다.
- 일부 자주 사용되는 대형 쿼리는 속도를 위해 SP를 쓰는등 별도의 튜닝이 필요한 경우가 있다.
- DBMS의 고유 기능을 이용하기 어렵다. (하지만 이건 단점으로만 볼 수 없다 : 특정 DBMS의 고유기능을 이용하면 이식성이 저하된다.)
- 프로시저가 많은 시스템에선 ORM의 객체 지향적인 장점을 활용하기 어렵다.
- 이미 프로시저가 많은 시스템에선 다시 객체로 바꿔야하며, 그 과정에서 생산성 저하나 리스크가 많이 발생할 수 있다.
