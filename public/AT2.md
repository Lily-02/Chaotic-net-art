---
title: Assignment 🕊
published_at: 2024-03-06
snippet: Chaotic aesthetic from fractal, glitch and post-digital net art
disable_html_sanitization: true

---

<!-- Audio sound -->
<audio id="backgroundMusic" loop>
	<source src="code.mp3" type="23/code.mp3">
    Your browser does not support the audio element.
</audio>
<!-- Canvas and Audio Button -->
<div style="position: relative; display: inline-block;">
	<canvas id="c2_example" width="300" height="300" style="border: 1px solid black;"></canvas>
	<button id="playButton" style="position: absolute; top: 10px; left: 10px;">Play Background Audio</button>
</div>
<!-- Include c2.js library files -->
<script src="/script/c2.min.js"></script>
<script src="/script/c2.js"></script>
<script>
// Set up c2 renderer
cnv = document.getElementById('c2_example')
const renderer = new c2.Renderer(cnv)
resize()
renderer.background ('#cccccc')
let random = new c2.Random ()
// Play background audio
var audio = document.getElementById('backgroundMusic');
var playButton = document.getElementById('playButton');
// Setting button style properties
playButton.style.display = 'inline-block'; // Makes the button only as wide as necessary
playButton.style.padding = '8px 16px'; // Adds some padding around the text
playButton.style.fontSize = '16px'; // Adjust the font size as needed
playButton.style.cursor = 'pointer'; // Shows a pointer cursor on hover
playButton.addEventListener('click', function() {
	audio.play();
});
// Cell class
class Agent extends c2.Cell{
	constructor(x, y, vx, vy) {
		x = x !== undefined ? x : random.next(renderer.width);				// x-position of the cell
		y = y !== undefined ? y : random.next(renderer.height);				// y-position of the cell
		let r = random.next(renderer.width / 80, renderer.width / 30);		// radius of the cell
		super(x, y, r);
		this.vx = vx !== undefined ? vx : random.next(-2, 2);		// x-velocity
		this.vy = vy !== undefined ? vy : random.next(-2, 2);		// y-velocity
		this.color = c2.Color.hsl(random.next(0, 360), random.next(0, 100), random.next(0, 100));		// cell's colour
	}
	// Get x-position of the cell
	getX() {
		return this.p.x;
	}
	// Get y-position of the cell
	getY() {
		return this.p.y;
	}
	// Update a cell's velocities and change its direction when it hits a wall
	update(){
		this.p.x += this.vx;		// Update new x-position based on the old x-position and x-velocity
		this.p.y += this.vy;		// Update new y-position based on the old y-position and y-velocity
		let is_collision = 0;		// Flag for collision (to handle the generation of fractal smoothly)
		if (this.p.x < 0) {			// If the cell hits the left wall
			this.p.x = 0;
			this.vx *= -1;
			is_collision = 1;
		} else if (this.p.x > renderer.width) {		// If the cell hits the right wall
			this.p.x = renderer.width;
			this.vx *= -1;
			is_collision = 3;
		}
		if (this.p.y < 0) {			// If the cell hits the top wall
			this.p.y = 0;
			this.vy *= -1;
			is_collision = 2;
		} else if (this.p.y > renderer.height) {	// If the cell hits the bottom wall
			this.p.y = renderer.height;
			this.vy *= -1;
			is_collision = 4;
		}
		return is_collision
	}
	// Draw the cell
	display(){
		if (this.state != 2) {
			// Draw fractal circles for each cell
			this.drawFractal(this.p.x, this.p.y, this.r);
		}
	}
	// Recursive function to draw fractal shapes based on the cell's position
	drawFractal(x, y, radius, depth = 3) {
		// Base case: stop recursion if the depth is zero or radius is too small
		if (depth === 0 || radius < 2) return;
		// Set the styling for the fractal circles
		renderer.stroke('#333');
		renderer.lineWidth(1);
		renderer.fill(this.color);
		// Draw the circle at the current position with the current radius
		renderer.circle(x, y, radius);
		// Calculate the new radius for the smaller circles
		let newRadius = radius * 0.6;
		// Recursively draw smaller circles to the East
		this.drawFractal(x + radius, y, newRadius, depth - 1);
		// Recursively draw smaller circles to the West
		this.drawFractal(x - radius, y, newRadius, depth - 1);
		// Recursively draw smaller circles to the North
		this.drawFractal(x, y + radius, newRadius, depth - 1);
		// Recursively draw smaller circles to the South
		this.drawFractal(x, y - radius, newRadius, depth - 1);
	}
}
// Array to hold the cells
let agents = new Array(1);
agents[0] = new Agent();		// Initialise the first cell
// Variables for glitch
let img_data					// URL data of the current canvas
let is_glitching = false		// Flag to switch between glitch and not glitch
let glitch_img = false			// URL data with some chunks removed
let count = 0 					// Frame number count
renderer.draw(async () => {
	let voronoi = new c2.LimitedVoronoi();
	voronoi.compute(agents);
	for (let i = 0; i < agents.length; i++) {
		agents[i].display();							// Display agent_i
		let is_collision = agents[i].update();			// Update agent_i's velocities and check for collisions
		if (is_collision && agents.length < 80) {		// If collided, add a new agent
			let x = agents[i].getX();
			let y = agents[i].getY();
			if (is_collision == 1){						// If collided on the left wall
				vx = random.next(0, 2)
				vy = random.next(-2, 2)
			}
			else if (is_collision == 3){				// If collided on the right wall
				vx = random.next(-2, 0)
				vy = random.next(-2, 2)
			}
			else if (is_collision == 2){				// If collided on the top wall
				vx = random.next(-2, 2)
				vy = random.next(0, 2)
			}
			else if (is_collision == 4){				// If collided on the bottom wall
				vx = random.next(-2, 2)
				vy = random.next(-2, 0)
			}
			let agent = new Agent(x, y, vx, vy);		// Create the new agent
			agents.push(agent);							// Add the agent to the array
		}
	}
	// Increase frame count
	count += 1
	// Add glitch every 1000 frames
	if (count % 1000 == 0){
		img_data = await renderer.canvas.toDataURL("image/jpeg")
		add_glitch()
	}
	// Set is_glitching
	const prob = is_glitching ? 0.05 : 0.02		// Threshold for toggling is_glitching
	if (Math.random () < prob) {				
		is_glitching = !is_glitching			// Toggle is_glitching if random < threshold
	}
	// Apply glitch
	if (is_glitching && glitch_img) draw(glitch_img)
})
// Draw an image
const draw = i => renderer.context.drawImage (i, 0, 0, cnv.width, cnv.height)
// Add glitch
const add_glitch = () => {
	const i = new Image()
	i.onload = () => {
		glitch_img = i
	}
	i.src = glitchify(img_data, 96, 6)		// Get the current URL image data and apply glitch on it
}
// Generate random integer
const rand_int = max => Math.floor (Math.random () * max)
// Glitchify
const glitchify = (data, chunk_max, repeats) => {
	const chunk_size = rand_int (chunk_max / 4) * 4 								// Random chunk size to remove
	const i = rand_int (data.length - 24 - chunk_size) + 24 						// Beginning index of the chunk to be removed
	const front = data.slice (0, i)													// Before the chunk
	const back = data.slice (i + chunk_size, data.length)							// After the chunk
	const result = front + back 													// Remove the chunk and concatenate the "before" and "after" parts
	return repeats == 0 ? result : glitchify (result, chunk_max, repeats - 1)		
}
// Style to fit the canvas to the viewport
window.addEventListener('resize', resize);
function resize() {
	renderer.size(window.innerWidth, window.innerHeight);
	document.documentElement.style.margin = '0';
	document.documentElement.style.padding = '0';
	document.documentElement.style.overflow = 'hidden';
	document.documentElement.style.height = '100%';
	document.body.style.margin = '0';
	document.body.style.padding = '0';
	document.body.style.overflow = 'hidden';
	document.body.style.height = '100%';
}
</script>