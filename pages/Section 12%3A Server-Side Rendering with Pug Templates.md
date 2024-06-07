- [[Section 12: Server-Side Rendering with Pug Templates/Server-Side vs Client-Side Rendering]]
- [[Section 12: Server-Side Rendering with Pug Templates/Setting up Pug in Express]]
- First Steps with Pug
  collapsed:: true
	- base.pug
	  collapsed:: true
		- ```javascript
		  doctype html
		  html
		      head
		          title Natours | #{tour}
		          link(rel='stylesheet' href='css/style.css')
		          link(rel='shortcut icon' type='image/png' href='img/favicon.png')
		      body
		          h1= tour
		          h2= user.toUpperCase()
		          // h1 The Park Camper
		  
		          - const x = 9;
		          h2= 2 * x
		          p This is just some text
		  ```
	- app.js
	  collapsed:: true
		- ```javascript
		  app.get('/', (req, res) => {
		    res.status(200).render('base', {
		      tour: 'The Forest Hiker',
		      user: 'Jonas',
		    });
		  });
		  ```
- Creating Our Base Template
- Including Files into Pug Templates
- Building the Login Screen