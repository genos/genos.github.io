+++
title = "Serpent Ski"
+++

# Sierpiński Triangle via Chaos

Short one today, as my [new job](https://ionq.com/) is keeping me busy.
Here's something fun:

1. Start with an equilateral triangle.
2. Pick a random point inside that triangle.
3. Pick one of the three vertices and move halfway towards it.
4. Make a mark.
5. Repeat 3 & 4.

You end up with the Sierpiński Triangle by way of the awesomely-named [Chaos
Game](https://en.wikipedia.org/wiki/Chaos_game).

# Demo

Borrowing how to calculate a random point inside a triangle from [this
reference](https://stackoverflow.com/questions/19654251) (remember when people
used StackOverflow instead of LLMs?), here's a demo.
Hit _Start/Stop_ to, well, start or stop, then _Reset_ to start over.
It'll probably slow down once there are a _lot_ of points, so caveat emptor.

<div style="display:flex;gap:10%">
<button type="button" id="start_stop" style="height:2.5rem">Start/Stop</button>
<button type="button" id="reset" style="height:2.5rem">Reset</button>
<svg viewBox="-2 -2 200 200" xmlns="http://www.w3.org/2000/svg" id="serpent_ski">
  <circle id="ss_a" cx="0" cy="100" r="1" fill="#027f8d" />
  <circle id="ss_b" cx="50" cy="13.397" r="1" fill="#027f8d" />
  <circle id="ss_c" cx="100" cy="100" r="1" fill="#027f8d" />
</svg>
</div>
<script>
const[e,t,a,l]=["serpent_ski","ss_a","ss_b","ss_c"].map(e=>document.getElementById(e)),n=t.getAttribute("fill"),c=[t,a,l].map(e=>e.cx.baseVal.value),r=[t,a,l].map(e=>e.cy.baseVal.value);let[s,d,o]=[0,0,null];const m=()=>{const[e,t]=[Math.sqrt(Math.random()),Math.random()];s=(1-e)*c[0]+e*(1-t)*c[1]+e*t*c[2],d=(1-e)*r[0]+e*(1-t)*r[1]+e*t*r[2]};document.getElementById("reset").addEventListener("click",()=>{o=clearInterval(o),e.replaceChildren(t,a,l),m()}),document.getElementById("start_stop").addEventListener("click",()=>{o?o=clearInterval(o):(m(),o=setInterval(()=>{const t=document.createElementNS("http://www.w3.org/2000/svg","circle"),a=Math.floor(c.length*Math.random());s+=.5*(c[a]-s),d+=.5*(r[a]-d),[["cx",s],["cy",d],["r",.5],["fill",n],["opacity",1]].forEach(([e,a])=>{t.setAttribute(e,a)}),e.appendChild(t)}))});
</script>

# Code plz

My Javascript code is short enough to include here; even unminified, it's only
43 lines after formatting with [`biome`](https://biomejs.dev/).[^1]

```javascript
const [S, A, B, C] = ["serpent_ski", "ss_a", "ss_b", "ss_c"].map((i) =>
	document.getElementById(i),
);
const FILL = A.getAttribute("fill");
const XS = [A, B, C].map((c) => c.cx.baseVal.value);
const YS = [A, B, C].map((c) => c.cy.baseVal.value);
let [X, Y, ON] = [0, 0, null];
const new_xy = () => {
	const [u, v] = [Math.sqrt(Math.random()), Math.random()];
	X = (1 - u) * XS[0] + u * (1 - v) * XS[1] + u * v * XS[2];
	Y = (1 - u) * YS[0] + u * (1 - v) * YS[1] + u * v * YS[2];
};
document.getElementById("reset").addEventListener("click", () => {
	ON = clearInterval(ON);
	S.replaceChildren(A, B, C);
	new_xy();
});
document.getElementById("start_stop").addEventListener("click", () => {
	if (!ON) {
		new_xy();
		ON = setInterval(() => {
			const p = document.createElementNS(
				"http://www.w3.org/2000/svg",
				"circle",
			);
			const i = Math.floor(XS.length * Math.random());
			X += 0.5 * (XS[i] - X);
			Y += 0.5 * (YS[i] - Y);
			[
				["cx", X],
				["cy", Y],
				["r", 0.5],
				["fill", FILL],
				["opacity", 1],
			].forEach(([k, v]) => {
				p.setAttribute(k, v);
			});
			S.appendChild(p);
		});
	} else {
		ON = clearInterval(ON);
	}
});
```

[^1]: There are also some lines of HTML needed to set up the buttons and SVG to begin with.
