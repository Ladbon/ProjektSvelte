<script lang="ts">
	import { onMount } from 'svelte';
	import jsPDF from 'jspdf';

	// Canvas properties
	let canvas: HTMLCanvasElement;
	let ctx: CanvasRenderingContext2D;

    // Create a new jsPDF instance
	const pdf = new jsPDF();

	// Images and text boxes on the canvas
	const elements: {
		type: 'image' | 'text';
		x: number;
		y: number;
		content: HTMLImageElement | string;
	}[] = [];

	// Current element being dragged
	let currentElement: {
		type: 'image' | 'text';
		x: number;
		y: number;
		content: HTMLImageElement | string;
	} | null = null;
	let offsetX: number = 0;
	let offsetY: number = 0;
	let isDragging: boolean = false;

	// Function to add an image to the canvas
	function addImage(imageUrl: string, x: number, y: number) {
		const image = new Image();
		image.src = imageUrl;
		image.onload = () => {
			ctx.drawImage(
				image,
				x,
				y,
				image.width, // Draw the image at its actual width
				image.height // Draw the image at its actual height
			);
			elements.push({ type: 'image', x, y, content: image });
		};
	}

	// Function to add a text box to the canvas
	function addTextBox(text: string, x: number, y: number) {
		ctx.font = '16px Arial'; // You can customize the font size and style
		const textWidth = ctx.measureText(text).width;
		const textHeight = 16; // Adjust this value for the desired text box height

		ctx.fillText(text, x, y + textHeight);
		elements.push({ type: 'text', x, y, content: text });
	}

	// Function to check if the mouse is over an element
	function isMouseOverElement(
		mouseX: number,
		mouseY: number,
		elem: { type: 'image' | 'text'; x: number; y: number; content: HTMLImageElement | string }
	) {
		if (elem.type === 'image') {
			return (
				mouseX >= elem.x &&
				mouseX <= elem.x + elem.content.width &&
				mouseY >= elem.y &&
				mouseY <= elem.y + elem.content.height
			);
		} else {
			// Assuming a fixed text height for simplicity
			const textWidth = ctx.measureText(elem.content as string).width;
			const textHeight = 16; // Adjust this value for the desired text box height
			return (
				mouseX >= elem.x &&
				mouseX <= elem.x + textWidth &&
				mouseY >= elem.y &&
				mouseY <= elem.y + textHeight
			);
		}
	}

	// Function to handle mouse down event
	function handleMouseDown(event: MouseEvent) {
		const mouseX = event.clientX - canvas.getBoundingClientRect().left;
		const mouseY = event.clientY - canvas.getBoundingClientRect().top;

		// Check if we clicked on an element (image or text)
		for (const elem of elements) {
			if (isMouseOverElement(mouseX, mouseY, elem)) {
				currentElement = elem;
				offsetX = mouseX - elem.x;
				offsetY = mouseY - elem.y;
				isDragging = true;
				break;
			}
		}
	}

	// Function to handle mouse move event
	function handleMouseMove(event: MouseEvent) {
		if (isDragging && currentElement) {
			const mouseX = event.clientX - canvas.getBoundingClientRect().left;
			const mouseY = event.clientY - canvas.getBoundingClientRect().top;

			// Calculate the new element position relative to the mouse
			const newX = mouseX - offsetX;
			const newY = mouseY - offsetY;

			// Ensure the element stays within the canvas boundaries
			const canvasRightBound =
				canvas.width -
				(currentElement.type === 'image'
					? currentElement.content.width
					: ctx.measureText(currentElement.content as string).width);
			const canvasBottomBound =
				canvas.height - (currentElement.type === 'image' ? currentElement.content.height : 16); // Assuming fixed text height

			if (
				newX >= 0 && // Check left boundary
				newX <= canvasRightBound && // Check right boundary
				newY >= 0 && // Check top boundary
				newY <= canvasBottomBound // Check bottom boundary
			) {
				currentElement.x = newX;
				currentElement.y = newY;
				redrawCanvas();
			}
		}
	}

	// Function to handle mouse up event
	function handleMouseUp() {
		isDragging = false;
		currentElement = null;
	}

	// Function to redraw the canvas
	function redrawCanvas() {
		ctx.clearRect(0, 0, canvas.width, canvas.height);
		for (const elem of elements) {
			if (elem.type === 'image') {
				ctx.drawImage(
					elem.content as HTMLImageElement,
					elem.x,
					elem.y,
					elem.content.width,
					elem.content.height
				);
			} else {
				ctx.font = '16px Arial'; // You can customize the font size and style
				ctx.fillText(elem.content as string, elem.x, elem.y + 16); // Assuming fixed text height
			}
		}
	}

	onMount(() => {
		// Initialize canvas
		canvas = document.getElementById('canvas') as HTMLCanvasElement;
		ctx = canvas.getContext('2d')!;

		// Add event listeners
		canvas.addEventListener('mousedown', handleMouseDown);
		canvas.addEventListener('mousemove', handleMouseMove);
		canvas.addEventListener('mouseup', handleMouseUp);

		// Add cat images as prefabs
		addImage('https://ps.w.org/ai-engine/assets/icon-128x128.png?rev=2839659', 25, 25);
	});

	// Function to handle canvas click (add text box)
	function handleCanvasClick(event: MouseEvent) {
		const mouseX = event.clientX - canvas.getBoundingClientRect().left;
		const mouseY = event.clientY - canvas.getBoundingClientRect().top;

		// Check if we clicked on the canvas (not on an existing element)
		let clickedOnElement = false;
		for (const elem of elements) {
			if (isMouseOverElement(mouseX, mouseY, elem)) {
				clickedOnElement = true;
				break;
			}
		}

		if (!clickedOnElement) {
			const text = prompt('Enter text for the text box:');
			if (text) {
				addTextBox(text, mouseX, mouseY);
				redrawCanvas();
			}
		}
	}

	// Add click event listener to the canvas
	onMount(() => {
		canvas.addEventListener('click', handleCanvasClick);
	});

    function exportToPDF() {
    // Create a new jsPDF instance
    const pdf = new jsPDF();

    // Get the canvas data URL
    const canvasDataURL = canvas.toDataURL('image/png');

    // Add an image to the PDF
    pdf.addImage(canvasDataURL, 'PNG', 10, 10, 100, 100); // Adjust the position and size as needed

    // Save the PDF
    pdf.save('canvas.pdf');
  }
</script>

<!-- Canvas -->
<canvas id="canvas" width="1300" height="700" />

<!-- Export button -->
<button on:click={exportToPDF}>Export to PDF</button>

<style>
  canvas {
    border: 1px solid #000;
  }
</style>