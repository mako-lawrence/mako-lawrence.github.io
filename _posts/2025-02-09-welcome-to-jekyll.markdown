---
layout: post
title:  "Developing task management application"
date:   2025-02-09 18:50:54 -0600
categories: jekyll update
---

This post introduces web application that I developed for my senior project.  

# Backgrounds #  
I chose to develop a task management application for two main reasons.

### From a Learning Perspective ###  
The task management application is simple in functionality, yet it comprehensively covers the fundamental components required for real-world application development (such as backend CRUD operations, frontend form handling, and database integration). This made it an ideal project with a realistic level of difficulty that matched my current skill set. Although I had other options, such as a note-taking app, an app with posting features, or a schedule management app, I ultimately chose to develop a task management app because of the second reason.

### Based on Real-Life Demand and Personal Experience ###  
At this university, I often heard that students were either pursuing double majors or facing a heavy load of assignments. I, too, frequently struggled with managing my tasks and assignments. Given this background, I felt there was a need for a tool to help organize personal tasks and assignments, and I wanted to create an application that could assist students in managing their daily workload.

# Objectives #  
I had two main objectives for this senior project. First, I wanted to study the fundamental skills required for application development. Second, I aimed to learn the entire process from development to deployment. I am still not very familiar with programming and it was the first time to develop and deploy an application from scratch, because I switched my major from economics to computer science after transferring from a Japanese university. This project provided my first opportunity to develop and deploy an application from scratch, allowing me to build my foundational skills and gain hands-on experience throughout the entire development cycle.

# Used Technologies #  
### Python(flask) ###
The backend functions are handled using the Python Flask framework, which is primarily used to implement the following features:  
-Launching the application  
-User management functions such as login and user registration  
-Interactions with the database  
-Implementation of basic CRUD operations (Create, Read, Update, Delete)  
-Automatic task sorting  

### HTML/CSS(Jinja2) ###  
Used for the front-end to manage the application’s appearance and to retrieve and display task information provided by the backend using Jinja2.

### AWS EC2 ###  
Used for infrastructure purposes. It was utilized to set up the deployment server via an SSH connection.

# About Backend #  
As mentioned in the technology section, I used Flask—a Python framework—for the backend. Flask is a micro web framework for Python that is characterized by its lightweight and simple design, making it well-suited for developing small to medium-sized web applications. Therefore, Flask was the ideal choice for a web application with simple and basic functionalities like this task management app.
Using the following “Create Task” code as an example, I will explain how the backend is structured and how functionality is processed using Flask.  
```
# Create a new task
@bp.route('/create_task', methods=['GET', 'POST'])
# Can be executed when logged in
@login_required
def create_task():
    # In case of a POST request
    if request.method == 'POST':        
        # Retrieve the submitted items
        title = request.form.get('title')
        detail = request.form.get('detail')
        end_time = datetime.strptime(request.form.get('end_time'), '%Y-%m-%d')
        
        # If any field is left blank
        if title == '' or detail =='' or end_time == '':
            flash('There is an empty form')
        # If all inputs are correct
        else:
            # Map the retrieved items to the column names in the database
            create_task = Task(
                title = title,
                end_time = end_time,
                detail = detail,
                user_id = current_user.get_id()
                )
            
            # Write to the database (Create operation)
            try:
                with db.session.begin(subtransactions=True):
                    db.session.add(create_task)
                db.session.commit()
            except:
                db.session.rollback()
                raise
            finally:
                db.session.close()
                
            return redirect(url_for('todo_app.user'))
    return render_template('create_task.html')
```

### Function Invocation ###  
In Flask, you define which function should be invoked for a specific URL path by using decorators such as @app.route() or @bp.route().
```
# Create a new task
@bp.route('/create_task', methods=['GET', 'POST'])
def create_task():
```
### Login Verification ###  
```
@login_required
```
This is a decorator provided by the Flask-Login extension, which restricts access to this endpoint to only those users who are logged in. 
Flask-Login simplifies user authentication and session management.

### Request Method Check ###  
```
if request.method == 'POST':
```
In Flask, you can access HTTP request information (such as the method, form data, cookies, etc.) through the request object. 
This allows you to determine whether the user’s request is a GET or POST request and process the form data accordingly.

### Page Redirection ###  
```
return redirect(url_for('todo_app.user'))
```
The functions redirect() and url_for() are helper functions in Flask. url_for() dynamically generates a URL for an endpoint within the application, and redirect() sends the user to the specified URL. This ensures that after the processing is completed, the user is directed to the appropriate page.

# About Frontend #
While the backend handles the business logic and database interactions, the frontend determines how data is presented and how users interact with it. In this task management application, I used HTML and CSS along with Flask’s Jinja2 templating engine to generate dynamic content.

### Templating with Jinja2 ###
Jinja2 is a powerful templating engine that comes with Flask. By using Jinja2, the application can:
  •Insert dynamic data: Display real-time information about tasks, such as titles, due dates, and details, directly in HTML using the syntax {% raw %}{{ ... }}{% endraw %}.
  •Control page flow: Employ conditional structures ({% raw %}{% if %}) {% endraw %}or loops {% raw %}({% for %}) {% endraw %}within the HTML to selectively render expired tasks, list all tasks, or handle user-specific content.
  •Template inheritance: Define a base.html with shared elements like <head>, navigation bars, and footer, then let other pages (create_task.html, user.html, etc.) extend base.html with {% raw %}{% extends 'base.html' %}{% endraw %}. Each page only needs to supply the unique block content, avoiding code duplication.

This simplifies updates, because common sections of the layout are centralized in one template.

### Frontend Structure ###
Using the following code used in "base.html" and "user.html" which are used in the user main page to display tasks as an example, I will explain the frontend structure.

Here is base.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    
    {% block head %}
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>
            {% block title %}
            {% endblock title %}
        </title>
        <link rel="preconnect" href="https://fonts.googleapis.com">
        <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
        <link href="https://fonts.googleapis.com/css2?family=BIZ+UDPGothic&display=swap" rel="stylesheet">
        <link rel="stylesheet" href="{{url_for('static',filename='css/reset.css')}}">
        <link rel="stylesheet" href="{{url_for('static',filename='css/styles.css')}}">
        {% endblock head %}
        
</head>
<body>
    <div class="wrapper">
        <div id="header">
            <ul id="nav">
                <li><a class="nav-links" href="{{url_for('todo_app.home')}}">Home</a></li>
                {% if current_user.is_authenticated %}
                <li><a class="nav-links" aria-current="page" href="{{ url_for('todo_app.logout') }}">Logout</a></li>
                {% endif %}
                <li><a class="nav-links" href="{{ url_for('todo_app.user') }}">User Page</a></li>
            </ul>
        </div>
        {% block content %}
        {% endblock content %}
    </div>
    </body>
</html>
```

and user.html
```
{% from "_formhelpers.html" import render_field %}
{% extends 'base.html' %}

{% block title %}Task List Page{% endblock %}

{% block content %}
<main>
    <div class="container">
        <div class="task-add-button-container">
            <button class="blue-button task-add-button">
                <a class="" href="/create_task" role="button">Create New</a>
            </button>
        </div>
        <div class="tasks-wrapper">
            {% for task in tasks %}
            <div class="task-container">
                <div class="task-detail">
                    <h2 class="task-title">{{ task.title }}</h2>
                    <p>Deadline: {{ task.end_time.date() }}</p>
                    {% if task.end_time.date() < today %}
                    <button class="orange-button expired-button">
                        Expired!
                    </button>
                    {% endif %}
                    
                </div>
                <div class="task-edit-buttons-container">
                    <button class="gray-button"><a href="/detail/{{ task.id }}" role="button">Details</a></button>
                    <button class="green-button"><a href="/update/{{ task.id }}" role="button">Edit</a></button>
                    <button class="red-button"><a href="/delete/{{ task.id }}" role="button">Complete</a></button>
                </div>
            </div>
            {% endfor %}
        </div>
    </div>
</main>
{% endblock content %}
```

### Navigation and Conditional Rendering ###
Jinja2 allows for control structures like {% raw %}{% if current_user.is_authenticated %}{% endraw %} to show or hide links depending on whether the user is logged in.
URLs for different routes are constructed using url_for('route_name').

### Dynamic Content Display ###
The application passes a list of task objects (e.g., tasks) from the Flask backend to a template like user.html.
Using {% raw %}{% for task in tasks %}{% endraw %}, each task’s details (title, deadline, etc.) are displayed.
A comparison like {% raw %}if task.end_time.date() < today{% endraw %} triggers an “Expired!” button to appear.

### User Interaction ###
Buttons link to CRUD routes: “Create New” (/create_task), “Edit” (/update/<task_id>), “Complete” (/delete/<task_id>), etc.
By placing these buttons prominently, users can easily create, view, and edit tasks from the front end.

Overall, Flask + Jinja2 provides a straightforward way to pass data (like tasks) from the backend to the frontend, handle user actions through forms or links, and dynamically render HTML based on real-time application data.

# Demonstration #
Now, I’m gonna show you what the application looks like.
![home]({{ site.baseurl }}/images/scr-home.png)
This is the Task management application. 
![login]({{ site.baseurl }}/images/scr-login.png)
![register]({{ site.baseurl }}/images/scr-register.png)
When you open the application, you’ll see the login and register page.
you need to enter your name, email and passwordand and click the register button to login in register page.
Then, you should be able to log in
![main]({{ site.baseurl }}/images/scr-userpage.png)
After you log-in, you'll see the user main page. To create a new task, click the "Create New" button in the upper left corner of the user page.
![task]({{ site.baseurl }}/images/scr-createtask.png)
You can enter the task title, details, and due date to create a task.
![displaytasks]({{ site.baseurl }}/images/scr-sorted-expired.png)
After you create your task, it appears on the user page with its title and due date.  
When you want to see your task detail, click the gray detail button and then you can see it and you can edit your task from the green edit button if you have some mistakes or anything that you want to change.  
If you create two ore more tasks, they are automatically sorted by due date like the image, which makes it easier to prioritize which ones to work on first.  
Moreover, when a task's deadline passes, "Expired" is displayed like the task at the top, so you can quickly notice tasks that have passed their deadlines.  
Finally, once you complete a task and wish to remove it, simply push the red "Complete" button.   
