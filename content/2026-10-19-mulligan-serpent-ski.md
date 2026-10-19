+++
title = "Serpent Ski Mulligan"
+++

# Gotta go fast

The approach I used [last time](@/2026-10-15-serpent-ski.md) isn't very
efficient, since it repeatedly appends new SVG elements.
Here's a speedier approach, via drawing on a `canvas` 2D context:

<div style="display:flex;gap:10%">
<button type="button" id="start_stop" style="height:2.5rem">Start/Stop</button>
<button type="button" id="reset" style="height:2.5rem">Reset</button>
<canvas id="chaos" width="400" height="400">
  Sierpiński Triangle via the Chaos Game
</canvas>
</div>
<script>
const t=document.getElementById("chaos").getContext("2d");t.fillStyle="#027f8d";const[e,a]=(e=>{const a=.75*t.canvas.width,n=a*Math.sqrt(3)/2;return[[10,e+a/2,e+a],[e+n,10,e+n]]})(10);let[n,c,l]=[0,0,null];const r=()=>{const[t,l]=[Math.sqrt(Math.random()),Math.random()];n=(1-t)*e[0]+t*(1-l)*e[1]+t*l*e[2],c=(1-t)*a[0]+t*(1-l)*a[1]+t*l*a[2]},o=(e,a,n=2)=>{t.beginPath(),t.arc(e,a,n,0,2*Math.PI,!1),t.fill()};e.forEach((t,e)=>o(t,a[e],4)),document.getElementById("reset").addEventListener("click",()=>{l=clearInterval(l),t.clearRect(0,0,t.canvas.width,t.canvas.height),e.forEach((t,e)=>o(t,a[e],4))}),document.getElementById("start_stop").addEventListener("click",()=>{l?l=clearInterval(l):(r(),l=setInterval(()=>{const t=Math.floor(e.length*Math.random());n+=.5*(e[t]-n),c+=.5*(a[t]-c),o(n,c)}))});
</script>


The code is shorter, too:

```javascript
const CTX = document.getElementById("chaos").getContext("2d");
CTX.fillStyle = "#027f8d";
const [XS, YS] = ((b) => {
	const w = CTX.canvas.width;
	const s = 0.75 * w;
	const h = (s * Math.sqrt(3)) / 2;
	return [
		[b + 0, b + s / 2, b + s],
		[b + h, b + 0, b + h],
	];
})(10);
let [X, Y, ON] = [0, 0, null];
const new_xy = () => {
	const [u, v] = [Math.sqrt(Math.random()), Math.random()];
	X = (1 - u) * XS[0] + u * (1 - v) * XS[1] + u * v * XS[2];
	Y = (1 - u) * YS[0] + u * (1 - v) * YS[1] + u * v * YS[2];
};
const draw = (x, y, r = 2) => {
	CTX.beginPath();
	CTX.arc(x, y, r, 0, 2 * Math.PI, false);
	CTX.fill();
};
XS.forEach((x, i) => draw(x, YS[i], 4));
document.getElementById("reset").addEventListener("click", () => {
	ON = clearInterval(ON);
	CTX.clearRect(0, 0, CTX.canvas.width, CTX.canvas.height);
	XS.forEach((x, i) => draw(x, YS[i], 4));
});
document.getElementById("start_stop").addEventListener("click", () => {
	if (!ON) {
		new_xy();
		ON = setInterval(() => {
			const i = Math.floor(XS.length * Math.random());
			X += 0.5 * (XS[i] - X);
			Y += 0.5 * (YS[i] - Y);
			draw(X, Y);
		});
	} else {
		ON = clearInterval(ON);
	}
});
```
