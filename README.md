Crearemos una aplicación para mostrar la facilidad con la que podremos crear una aplicación rails con ajax. La aplicación la subiremos a heroku. Usaremos la siguiente configuración:

```sh
ruby 2.4.0p0 (2016-12-24 revision 57164) [x86_64-linux]
Rails 5.2.0
```

Creemos la aplicación:

```sh
> rails new app --database=postgresql
```

Agreguemos los respositorios:

```sh
> cd app/
> git init
> git add .
> git commit -m 'Primer commit'
> git remote add github https://github.com/davidmaogomezz/rails-ajax.git
> heroku git:remote -a railsajax
> git push github master
> git push heroku master
```

Agregue mi app a dos repositorios a github y a heroku. Ahora podemos coger los datos de la base de datos que heroku nos crea cada vez que desplegamos un proyecto y los agregamos en ```app/config/database.yml```.

```ruby
development:
  <<: *default
  database: ""
  pool: 5
  username: ""
  password: ""
  host:     ""
  sslmode:  require
```

Llegados a este punto podemos probar nuestra aplicación.

```sh
> rails s
```

Crearemos un scaffold User que nos servirá para ilustrar la implementación de ajax.

```sh
> rails g scaffold User name:string email:string birthday:date
> rake db:migrate
```

Ahora queremos que el root de nuestra aplicación sea el index de User. Entonces agregamos en ```app/config/routes.rb```

```ruby
root 'users#index'
```
Debemos agregar al gemfile

```ruby
gem 'jquery-rails', '~> 4.1', '>= 4.1.1'
```

Y en ```app/assets/javascripts/application.js```

```js
//= require jquery
```

```sh
> bundle install
```

Ahora en ```/app/app/views/users``` modficaremos el archivo ```_form.html.erb``` en la primer linea con 

```ruby
<%= form_with(model: user, remote: true) do |form| %>
```

Queremos llamar el form de User desde la vista index para ver el funcionamiendo de ajax.

```html
<p id="notice"><%= notice %></p>

<h1>Users</h1>

<%= render "form", user:@user%>

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Email</th>
      <th>Birthday</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @users.each do |user| %>
      <%= render 'row_user', user: user%>
    <% end %>
  </tbody>
</table>
```

Hemos creado la vista parcial con el fin de reutilizar código. ```_row_user.html.erb```

```html
<tr id="<%=user.id%>">
    <td><%= user.name %></td>
    <td><%= user.email %></td>
    <td><%= user.birthday %></td>
    <td><%= link_to 'Show', user %></td>
    <td><%= link_to 'Edit', edit_user_path(user) %></td>
    <td><%= link_to 'Destroy', user, method: :delete, data: { confirm: 'Are you sure?' }, :remote => true %></td>
</tr>
```

Y debemos crear el objeto @user en el método index

```ruby
  def index
    @user = User.new
    @users = User.all
  end
```

Ahora modificaremos el contraldor de User en el método create para que cuando creemos un usuario  la aplicación responda en formato js:

```ruby
  def create
    @user = User.new
    respond_to do |format|
      format.html
      format.js
    end    
  end
```

creamos el archivo ```create.js.erb``` en ```app/app/views/users``` desde acá cuando creemos un usuario es desde donde reutilizaremos la vista parcial ```_row_user.html.erb```

```ruby
$("#table-user > tbody").append("<%= j render 'row_user', user: @user %>");
```

Y ya tendremos el método create funcionando. Ahora vamos a agregarle ajax a la acción destroy. En el controlador User

```ruby
    @user.destroy
    respond_to do |format|
      format.html { redirect_to users_url, notice: 'User was successfully destroyed.' }
      format.json { head :no_content }
      format.js
    end
```

Anteriormente anteriormente el link_to de destroy le habiamos agregado ```:remote => true ``` con esto ya podremos crear el archivo ```destroy.js.erb```  con el siguiente conténido

```ruby
$("tr#<%= @user.id %>").slideUp();
```

Y ya tenemos la funcionalidad de agregar y eliminar usuarios con ajax.

[Acá puedes encontrar el respositorio de este proyecto](https://github.com/davidmaogomezz/rails-ajax).

[Y el demo de este proyecto lo puedes ver acá](https://railsajax.herokuapp.com/)