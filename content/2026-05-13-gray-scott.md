+++
title = "Gray-Scott Reaction-Diffusion"
[extra]
use_katex = true
+++

# On a world Turing

I was recently one of the [luck 10,000](https://xkcd.com/1053/) when I stumbled
across [these implementations](https://github.com/dlozeve/reaction-diffusion)
of [a Reaction-Diffusion
system](https://en.wikipedia.org/wiki/Reaction%E2%80%93diffusion_system) in APL
& BQN.
Investigating more, it turns out these ideas date back at least as far as [Alan
Turing](https://www.dna.caltech.edu/courses/cs191/paperscs191/turing.pdf).
Karl Sims's [particularly useful tutorial](https://www.karlsims.com/rd.html)
describes the Gray-Scott model.
In this, the system describes the interactions of two "chemicals" \\(A\\) and
\\(B\\) according to the following pair of differential equations:

\\[\frac{\partial A}{\partial t} = D_A\nabla^2A - AB^2 + f(1 - A)\\]
\\[\frac{\partial B}{\partial t} = D_B\nabla^2B + AB^2 - (k + f)B \\]

There are a
[number](https://github.com/EnvModelling/Env_modelling/blob/master/Spatio-temporal-modelling/Reaction_diffusion_2D.ipynb)
of great [resources](https://kaesve.nl/projects/reaction-diffusion/readme.html)
all over [the web](https://www.redblobgames.com/x/1905-reaction-diffusion/)
covering different ways of modeling this in code.

# Demo

Borrowing some ideas from the aforementioned sources, here's a little demo
implemented in Javascript.
Adjust the parameters to your heart's content, then hit _play/stop_ & _reset_
as desired!

<div style="display:flex;gap:10%;">
<div>
<label style="display:inline-block;width:5rem;" for="seed">Seed:</label><span> 1729</span><input type="range" id="seed" name="seed" value="1729" min="0" max="9999" oninput="this.previousElementSibling.innerText=this.value.padStart(4,'0').padStart(5,' ')"/></br>
<label style="display:inline-block;width:5rem;" for="da">D<sub>A</sub>:</label><span>0.420</span><input type="range" id="da" name="da" value="0.420" min="0" max="0.5" step="0.01" oninput="this.previousElementSibling.innerText=Number(this.value).toFixed(3)"/></br>
<label style="display:inline-block;width:5rem;" for="db">D<sub>B</sub>:</label><span>0.080</span><input type="range" id="db" name="db" value="0.080" min="0" max="0.5" step="0.01" oninput="this.previousElementSibling.innerText=Number(this.value).toFixed(3)"/></br>
<label style="display:inline-block;width:5rem;" for="feed">Feed Rate:</label><span>0.055</span><input type="range" id="feed" name="feed" value="0.055" min="0" max="0.1" step="0.001" oninput="this.previousElementSibling.innerText=Number(this.value).toFixed(3)"/></br>
<label style="display:inline-block;width:5rem;" for="kill">Kill Rate:</label><span>0.062</span><input type="range" id="kill" name="kill" value="0.062" min="0" max="0.1" step="0.001" oninput="this.previousElementSibling.innerText=Number(this.value).toFixed(3)"/></br>
<button type="button" id="render" style="width: 100%">Play</button>
<button type="button" id="reset"  style="width: 100%">Reset</button>
</div>
<canvas id="canvas"></canvas>
</div>
<script>
"use strict";var c=new Float64Array(65536),l=new Float64Array(65536),u=document.getElementById("canvas");u.width=u.height=256;var h=u.getContext("2d"),d=new ImageData(256,256),i={reset:document.getElementById("reset"),render:document.getElementById("render")},N=!1,x=t=>{let e=BigInt(t),a=(1n<<64n)-1n;return()=>{e=e+0x9e3779b97f4a7c15n&a;let r=e;return r=(r^r>>30n)*0xbf58476d1ce4e5b9n&a,r=(r^r>>27n)*0x94d049bb133111ebn&a,Number(r^r>>31n&a)/2**64}},m=t=>{let e=128,a=25,r=x(t);for(let o=0;o<256;o++)for(let s=0;s<256;s++){let n=o+s*256;o>=e-a&&o<e+a&&s>=e-a&&s<e+a?(c[n]=.5,l[n]=.25):c[n]=1-(l[n]=.25*r())}v()},b=(t,e)=>(t[e-256]??0)+(t[e+1]??0)+(t[e+256]??0)+(t[e-1]??0)-4*t[e],p=(t,e,a,r)=>{for(let o=0;o<256;o++)for(let s=0;s<256;s++){let n=o+s*256,f=c[n]*l[n]*l[n];c[n]+=t*b(c,n)-f+a*(1-c[n]),l[n]+=e*b(l,n)+f-(r+a)*l[n]}},v=()=>{for(let t=0;t<65536;t++){let e=16*Math.pow(l[t],2);d.data[4*t]=Math.floor(2*e),d.data[4*t+1]=Math.floor(127*e),d.data[4*t+2]=Math.floor(141*e),d.data[4*t+3]=255}h.putImageData(d,0,0)},g=()=>{p(Number(da.value),Number(db.value),Number(feed.value),Number(kill.value)),v(),N&&requestAnimationFrame(g)};m(Number(seed.value));i.reset.addEventListener("click",()=>{m(Number(seed.value))});i.render.addEventListener("click",()=>{N=!N,i.render.innerHTML=N?"Stop":"Play",N&&g()});
</script>

# Code plz

My Javascript code is short enough to include here; even unminified, it's only
75 lines after formatting with [`deno`](https://deno.com/).[^1]


```javascript
"use strict";
const N = 256;
const A = new Float64Array(N * N), B = new Float64Array(N * N);
const canvas = document.getElementById("canvas");
canvas.width = canvas.height = N;
const ctx = canvas.getContext("2d");
const pixels = new ImageData(N, N);
const buttons = {
  reset: document.getElementById("reset"),
  render: document.getElementById("render"),
};
let running = false;
const splitmix = (seed) => {
  let state = BigInt(seed);
  const mask = (1n << 64n) - 1n;
  return () => {
    state = (state + 0x9e3779b97f4a7c15n) & mask;
    let z = state;
    z = (z ^ (z >> 30n)) * 0xbf58476d1ce4e5b9n & mask;
    z = (z ^ (z >> 27n)) * 0x94d049bb133111ebn & mask;
    return Number(z ^ (z >> 31n) & mask) / 2**64;
  };
};
const init = (seed) => {
  const N2 = Math.floor(N / 2), R = Math.floor(N / 10);
  const rng = splitmix(seed);
  for (let x = 0; x < N; x++) {
    for (let y = 0; y < N; y++) {
      const i = x + y * N;
      if (x >= N2 - R && x < N2 + R && y >= N2 - R && y < N2 + R) {
        A[i] = 0.5;
        B[i] = 0.25;
      } else {
        A[i] = 1 - (B[i] = 0.25 * rng());
      }
    }
  }
  draw();
};
const laplacian = (m, i) => {
  return (m[i - N] ?? 0) + (m[i + 1] ?? 0) + (m[i + N] ?? 0) + (m[i - 1] ?? 0) -
    4 * m[i];
};
const step = (da, db, f, k) => {
  for (let x = 0; x < N; x++) {
    for (let y = 0; y < N; y++) {
      const i = x + y * N;
      const reaction = A[i] * B[i] * B[i];
      A[i] += da * laplacian(A, i) - reaction + f * (1 - A[i]);
      B[i] += db * laplacian(B, i) + reaction - (k + f) * B[i];
    }
  }
};
const draw = () => {
  for (let i = 0; i < N * N; i++) {
    const v = 16 * Math.pow(B[i], 2);
    pixels.data[4 * i] = Math.floor(2 * v);
    pixels.data[4 * i + 1] = Math.floor(127 * v);
    pixels.data[4 * i + 2] = Math.floor(141 * v);
    pixels.data[4 * i + 3] = 255;
  }
  ctx.putImageData(pixels, 0, 0);
};
const loop = () => {
  step(Number(da.value), Number(db.value), Number(feed.value), Number(kill.value));
  draw();
  if (running) requestAnimationFrame(loop);
};
init(Number(seed.value));
buttons.reset.addEventListener("click", () => { init(Number(seed.value)); });
buttons.render.addEventListener("click", () => {
  running = !running;
  buttons.render.innerHTML = running ? "Stop" : "Play";
  if (running) loop();
});
```


[^1]: I borrowed [this implementation](https://rosettacode.org/wiki/Pseudo-random_numbers/Splitmix64#JavaScript) of a fast, seedable PRNG.
