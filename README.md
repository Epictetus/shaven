# Shaven - Templating without mustaches!

Hey guys, look at present fasion... mustaches are not fashionable anymore =P.
Take a look how nice looking are shaven templates.

## Motivation

I'm not a designer, usualy all templates in my work are prepared by external
design studios or freelancers... But of course they are always pure xhtml.
So still we have to deal with them, convert to haml, fill in with mustaches or
erb sh**t! Now, my patience is over. Shaven will readmit some MVPC's fresh
air to your web apps and allow you to get rid of stupid logic from your views. 

## Installation

Installation with rubygems should go without quirks. Shaven depends on Nokogiri - if
you don't have it installed yet then i recommend you to check out its documentation
to avoid problems.

    $ gem install shaven

## How it works?

Shaven views are splited into two layers (similar to defunk's mustache) - Template
and Presenter. Templates are pure html files, Presenters are ruby classes
which provides data for templates. Depending on the data type provided by
presenter's methods you can freely and easily manipulate all contents within
your templates. Ok, lets finish talking and take a look at examples...

### Simple usage

    class SimplePresenter < Shaven::Presenter
      def title
      	"Hello world!"
      end

      def description
        "Yeah, hello beautiful code..."
      end
    end

    html = <<-HTML
    <!DOCTYPE html>
    <html>
    <head>
      <title rb="title">Example title!</title>
    </head>
    <body>
      <h1 rb="title">Example title</h1>
      <p rb="description">Example description...</p>
    </body>
    </html>
    HTML

    SimplePresenter.feed(html).to_html

This code produces following html:

    <!DOCTYPE html>
    <html>
    <head>
      <title>Hello World!</title>
    </head>
    <body>
      <h1>Hello World!</h1>
      <p>Yeah, hello beautiful code...</p>
    </body>
    </html>

### DOM manipulation

    class ManipulationPresenter < Shaven::Presenter
      # If you add parameter to the presenter method, original node will
      # passed to it. Given element is an Nokogiri::XML::Node with some
      # extra helpers for content manipulation. 
      def login_link(node)
        node.update!(:href => logout_path, :method => "delete")
      end

      # You can use extra html helpers to create new dom elements...
      def home_page_link
        a(:href => root_path, :class => "home-page-link") { "Go home!" }
      end
 
      # ... or to replace current.
      def title(node)
        node.replace! { tag(:h1, :id => "header") { "This is Sparta! "} }
      end
    end

    html = <<-HTML
    <!DOCTYPE html>
    <html>
    <body>
      <div rb="title">Example title</div>
      <a href="#" rb="logout_link">Logout!</a>
      <div rb="home_page_link">Home page link will go here...</div>
    </body>
    </html>
    HTML

    ManipulationPresenter.feed(html).to_html

Result:

    <!DOCTYPE html>
    <html>
    <body>
      <h1 id="header">This is Sparta!</h1>
      <a href="/logout" data-method="delete">Logout!</a>
      <div><a href="/" class="home-page-link">Go Home!</a></div>
    </body>
    </html>
    
### Hash scopes and lists

Now, the true power of Shaven. Suport for lists and scopes.

    class ComplexPresenter < Shaven::Presenter
      # As scopes are treaded all hashes and objects responding to `#to_shaven`
      # method (which returns hash with attributes).
      def user
        { :name => "John Doe",
          :email => "john@doe.com",
        }
      end

      def users_list
        [ { :name => tag(:strong) { "Emmet Brown" }, :email => "emmet@brown.com"},
          { :name => proc { |node| node.update!(:class => "marty") { "Marty Macfly" }, :email => "marty@macfly.com" },
          { :name => "Biff Tannen", :email => "biff@tannen.com" }
        ]
      end
    end

    html = <<-HTML
    <!DOCTYPE html>
    <html>
    <body>
      <h1>Single user here!</h1>
      <div rb="user">
        <h2 rb="name">Sample name</h2>
        <p rb="email">sapmle@email.com</p>
      </div>
      <h1>More users</h1>
      <ul id="users">
        <li rb="users_list">
          <span rb="name">Sample name</span>
          <span rb="email">sample@email.com</span>
        <li>
      </ul>
    </body>
    </html>
    HTML

And the awesome result is:

    <!DOCTYPE html>
    <html>
    <body>
      <h1>Single user here!</h1>
      <div rb="user">
        <h2>Adam Smith</h2>
        <p>adam@smith.com</p>
      </div>
      <h1>More users</h1>
      <ul id="users">
        <li>
          <span><strong>Emmet Brown</strong></span>
          <span>brown@brown.com</span>
        <li>
        <li class="marty">
          <span>Marty Macfly</span>
          <span>marty@macfly.com</span>
        <li>
        <li>
          <span>Biff Tannen</span>
          <span>biff@tannen.com</span>
        <li>
      </ul>
    </body>
    </html>

### Conditionals

    class ConditionalsPresenter < Shaven::Presenter
      def true?
        true
      end

      def false?
        false
      end
    end

    html = <<-HTML
    <!DOCTYPE html>
    <html>
    <body>
      <div rb:if="true?">Hello...</div>
      <div rb:unless="false?">World!</div>
    </body>
    </html>
    HTML
