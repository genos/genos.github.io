+++
title = "Zoomable Serpent Ski"
+++

# As you wish

As a [friend of mine](https://www.linkedin.com/in/jamesquantum/) pointed out, a
[fractal](@/2026-10-19-mulligan-serpent-ski.md) is more fun if you can zoom in on it!
Rather than handle dragging & zooming myself, I'll reach for [Konva](https://konvajs.org/).
A little longer this time, even with the help of [Konva's infinite canvas](https://konvajs.org/docs/sandbox/Infinite_Canvas.html):

```javascript
const STAGE = new Konva.Stage({
	container: "chaos",
	width: window.innerWidth,
	height: window.innerHeight,
	draggable: true,
});
const LAYER = new Konva.Layer();
STAGE.add(LAYER);
const [XS, YS] = ((b) => {
	const s = 0.75 * window.innerWidth;
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
const draw = (x, y, radius = 2) => {
	LAYER.add(
		new Konva.Circle({
			x,
			y,
			radius,
			fill: "#027f8d",
		}),
	);
};
XS.forEach((x, i) => draw(x, YS[i], 4));
document.getElementById("reset").addEventListener("click", () => {
	ON = clearInterval(ON);
	LAYER.destroyChildren();
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
STAGE.on("wheel", (e) => {
	e.evt.preventDefault();
	const dir = (e.evt.deltaY > 0 ? -1 : 1) * (e.evt.ctrlKey ? -1 : 1);
	const old = STAGE.scaleX();
	const scale = Math.max(0.1, Math.min(10, dir > 0 ? old * 1.05 : old / 1.05));
	STAGE.scale({ x: scale, y: scale });
	const p = STAGE.getPointerPosition();
	STAGE.position({
		x: p.x - scale * ((p.x - STAGE.x()) / old),
		y: p.y - scale * ((p.y - STAGE.y()) / old),
	});
});
```

<div style="display:flex-container;gap:10%">
  <button type="button" id="start_stop" style="height:2.5rem">Start/Stop</button>
  <button type="button" id="reset" style="height:2.5rem">Reset</button>
  <div id="chaos">
  </div>
</div>
<script src="https://unpkg.com/konva@10/konva.min.js"></script>
<script>const t=new Konva.Stage({container:"chaos",width:window.innerWidth,height:window.innerHeight,draggable:!0}),e=new Konva.Layer;t.add(e);const[n,a]=(t=>{const e=.75*window.innerWidth,n=e*Math.sqrt(3)/2;return[[10,t+e/2,t+e],[t+n,10,t+n]]})(10);let[o,r,d]=[0,0,null];const i=()=>{const[t,e]=[Math.sqrt(Math.random()),Math.random()];o=(1-t)*n[0]+t*(1-e)*n[1]+t*e*n[2],r=(1-t)*a[0]+t*(1-e)*a[1]+t*e*a[2]},l=(t,n,a=2)=>{e.add(new Konva.Circle({x:t,y:n,radius:a,fill:"#027f8d"}))};n.forEach((t,e)=>l(t,a[e],4)),document.getElementById("reset").addEventListener("click",()=>{d=clearInterval(d),e.destroyChildren(),n.forEach((t,e)=>l(t,a[e],4))}),document.getElementById("start_stop").addEventListener("click",()=>{d?d=clearInterval(d):(i(),d=setInterval(()=>{const t=Math.floor(n.length*Math.random());o+=.5*(n[t]-o),r+=.5*(a[t]-r),l(o,r)}))}),t.on("wheel",e=>{e.evt.preventDefault();const n=(e.evt.deltaY>0?-1:1)*(e.evt.ctrlKey?-1:1),a=t.scaleX(),o=Math.max(.1,Math.min(10,n>0?1.05*a:a/1.05));t.scale({x:o,y:o});const r=t.getPointerPosition();t.position({x:r.x-o*((r.x-t.x())/a),y:r.y-o*((r.y-t.y())/a)})});</script>
