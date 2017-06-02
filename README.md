# Snelly

Snelly is system for physically-based SDF (signed distance field) pathtracing in a web browser. 

The entire HTML source of a very basic scene is the following:

```javascript
<body onload="onLoad();">
<script type="text/javascript" src="../js/compiled/snelly.min.js"></script>
<script type="text/javascript">

function Scene() {}
Scene.prototype.init = function(snelly)
{
    this.par = {};
    this.par.R = 0.3;
    let materials = snelly.getMaterials();
    let metal = materials.loadMetal('Gold');
    metal.roughness = 0.05;
    let surface = materials.loadSurface();
    surface.roughness = 0.003;
    surface.diffuseAlbedo = [0.85, 0, 0];
    surface.specAlbedo = [0.16, 0.16, 0.16];
}

Scene.prototype.shader = function()
{
    return `
    uniform float R;
    float sdSphere(vec3 X, float r) { return length(X) - r; }   
    float sdBox(vec3 X, vec3 bmin, vec3 bmax)                     
    {                            
        vec3 d = abs(X-0.5*(bmin+bmax)) - 0.5*(bmax-bmin);
        return min(max(d.x,max(d.y,d.z)),0.0) + length(max(d,0.0));     
    } 

    float SDF_METAL(vec3 X)
    {
        X.xz = mod((X.xz),1.0)-vec2(0.5);
        return sdSphere(X, R);
    }

    float SDF_SURFACE(vec3 X)
    {            
        float L = 1.0e4;
        return sdBox(X, vec3(-L,-1.0,-L), vec3(L, 0.0,L));
    }
    `;
}

Scene.prototype.envMap = function() 
{ return 'https://cdn.rawgit.com/portsmouth/envmaps/74e9d389/HDR_112_River_Road_2_Bg.jpg'; }
Scene.prototype.initGui = function(gui) { gui.addSlider(this.par, {name: 'R', min: 0.0, max: 1.0}); }
Scene.prototype.syncShader = function(shader) { shader.uniformF("R", this.par.R); }

function onLoad() { snelly = new Snelly(new Scene()); animateLoop(); }
function animateLoop() { snelly.render(); window.requestAnimationFrame(animateLoop); }

</script>
</body>
```

