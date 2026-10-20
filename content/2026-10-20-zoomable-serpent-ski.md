+++
title = "Zoomable Serpent Ski"
+++

# As you wish

As a [friend of mine](https://www.linkedin.com/in/jamesquantum/) pointed out, a
[fractal](@/2026-10-19-mulligan-serpent-ski.md) is more fun if you can zoom in on it!
Rather than handle dragging & zooming myself, I'll reach for [Konva](https://konvajs.org/).
A little longer this time, even with the help of [Konva's infinite canvas](https://konvajs.org/docs/sandbox/Infinite_Canvas.html)
(_edit: especially when adding in_ [_pinch & zoom on mobile!_](https://konvajs.org/docs/sandbox/Multi-touch_Scale_Stage.html)):

```javascript
const STAGE = new Konva.Stage({
	container: "chaos",
	width: window.innerWidth,
	height: window.innerHeight,
	draggable: true,
});
Konva.hitOnDragEnabled = true;
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
const dist = (u, v) => {
	return Math.sqrt((v.x - u.x) ** 2 + Math.path(v.y - u.y, 2));
};
const center = (u, v) => {
	return { x: (u.x + v.x) / 2, y: (u.y + v.y) / 2 };
};
let [CENTER, DIST, STOPPED] = [null, 0, false];
STAGE.on("touchmove", (e) => {
	e.evt.preventDefault();
	const [t1, t2] = e.evt.touches.slice(0, 2);
	if (t1 && !t2 && !STAGE.isDragging() && STOPPED) {
		STAGE.startDrag();
		STOPPED = false;
	}
	if (t1 && t2) {
		if (STAGE.isDragging()) {
			STOPPED = true;
			STAGE.stopDrag();
		}
		const rect = STAGE.container().getBoundingClientRect();
		const p1 = {
			x: t1.clientX - rect.left,
			y: t1.clientY - rect.top,
		};
		const p2 = {
			x: t2.clientX - rect.left,
			y: t2.clientY - rect.top,
		};
		const c = center(p1, p2);
		if (!CENTER) {
			CENTER = c;
			return;
		}
		const d = dist(p1, p2);
		if (!DIST) {
			DIST = d;
		}
		const scale = STAGE.scaleX() * (d / DIST);
		STAGE.scaleX(scale);
		STAGE.scaleY(scale);
		const p = {
			x: (c.x - STAGE.x()) / STAGE.scaleX(),
			y: (c.y - STAGE.y()) / STAGE.scaleY(),
		};
		STAGE.position({
			x: c.x - p.x * scale + (c.x - CENTER.x),
			y: c.y - p.y * scale + (c.y - CENTER.y),
		});

		DIST = d;
		CENTER = c;
	}
});
STAGE.on("touchend", () => {
	DIST = 0;
	CENTER = null;
});
```

<div style="display:flex-container;gap:10%">
  <button type="button" id="start_stop" style="height:2.5rem">Start/Stop</button>
  <button type="button" id="reset" style="height:2.5rem">Reset</button>
  <div id="chaos">
  </div>
</div>
<script src="https://unpkg.com/konva@10/konva.min.js"></script>
<script>const t=new Konva.Stage({container:"chaos",width:window.innerWidth,height:window.innerHeight,draggable:!0});Konva.hitOnDragEnabled=!0;const e=new Konva.Layer;t.add(e);const[n,a]=(t=>{const e=.75*window.innerWidth,n=e*Math.sqrt(3)/2;return[[10,t+e/2,t+e],[t+n,10,t+n]]})(10);let[o,c,l]=[0,0,null];const i=()=>{const[t,e]=[Math.sqrt(Math.random()),Math.random()];o=(1-t)*n[0]+t*(1-e)*n[1]+t*e*n[2],c=(1-t)*a[0]+t*(1-e)*a[1]+t*e*a[2]},s=(t,n,a=2)=>{e.add(new Konva.Circle({x:t,y:n,radius:a,fill:"#027f8d"}))};n.forEach((t,e)=>s(t,a[e],4)),document.getElementById("reset").addEventListener("click",()=>{l=clearInterval(l),e.destroyChildren(),t.scaleX(1),t.scaleY(1),t.position({x:0,y:0}),n.forEach((t,e)=>s(t,a[e],4))}),document.getElementById("start_stop").addEventListener("click",()=>{l?l=clearInterval(l):(i(),l=setInterval(()=>{const t=Math.floor(n.length*Math.random());o+=.5*(n[t]-o),c+=.5*(a[t]-c),s(o,c)}))}),t.on("wheel",e=>{e.evt.preventDefault();const n=(e.evt.deltaY>0?-1:1)*(e.evt.ctrlKey?-1:1),a=t.scaleX(),o=Math.max(.1,Math.min(10,n>0?1.05*a:a/1.05));t.scale({x:o,y:o});const c=t.getPointerPosition();t.position({x:c.x-o*((c.x-t.x())/a),y:c.y-o*((c.y-t.y())/a)})});const r=(t,e)=>Math.sqrt((e.x-t.x)**2+Math.path(e.y-t.y,2)),d=(t,e)=>({x:(t.x+e.x)/2,y:(t.y+e.y)/2});let[h,y,x]=[null,0,!1];t.on("touchmove",e=>{e.evt.preventDefault();const[n,a]=e.evt.touches.slice(0,2);if(n&&!a&&!t.isDragging()&&x&&(t.startDrag(),x=!1),n&&a){t.isDragging()&&(x=!0,t.stopDrag());const e=t.container().getBoundingClientRect(),o={x:n.clientX-e.left,y:n.clientY-e.top},c={x:a.clientX-e.left,y:a.clientY-e.top},l=d(o,c);if(!h)return void(h=l);const i=r(o,c);y||(y=i);const s=t.scaleX()*(i/y);t.scaleX(s),t.scaleY(s);const g={x:(l.x-t.x())/t.scaleX(),y:(l.y-t.y())/t.scaleY()};t.position({x:l.x-g.x*s+(l.x-h.x),y:l.y-g.y*s+(l.y-h.y)}),y=i,h=l}}),t.on("touchend",()=>{y=0,h=null});</script>
