---
title: Colour picker
parent: Colours
nav_order: 4
---
<style>
    body {
        font-family: Arial, sans-serif;
        background: #f9f9f9;
        margin: 16px;
    }

    h1, h2, h3 {
        margin-top: 32px;
    }

    .palette-section {
        margin-bottom: 40px;
    }

    .palette-group {
        margin-bottom: 24px;
    }

    /* Responsive Grid */
    .palette {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(90px, 1fr));
        gap: 12px;
        width: 100%;
    }

    /* Color box */
    .color-box {
        --box-color: #000;
        background: var(--box-color);
        height: 110px;
        border-radius: 8px;
        display: flex;
        flex-direction: column;
        justify-content: flex-end;
        position: relative;
        box-shadow: 0 2px 6px rgba(0,0,0,.15);
        cursor: pointer;
        transition: transform .2s;
        border: none;
        text-align: left;
    }

    .color-box:hover {
        transform: scale(1.05);
    }

    .caption {
        background: rgba(255,255,255,.85);
        font-size: 12px;
        padding: 4px;
        border-radius: 0 0 8px 8px;
        text-align: center;
    }

    .caption strong {
        display: block;
    }

    .copied {
        position: absolute;
        top: 6px;
        right: 6px;
        background: rgba(0,0,0,.75);
        color: #fff;
        padding: 3px 6px;
        border-radius: 4px;
        font-size: 12px;
        opacity: 0;
        pointer-events: none;
        transition: opacity .2s;
    }

    .copied.visible {
        opacity: 1;
    }

    /* Controls */
    .controls {
        display: flex;
        flex-wrap: wrap;
        gap: 8px;
        margin: 8px 0 16px 0;
        font-size: 14px;
    }

    .controls label {
        font-weight: 700;
        min-width: 80px;
    }

    .steps {
        padding: 6px 8px;
        border-radius: 6px;
        border: 1px solid #ccc;
        background: #fff;
        min-width: 100px;
    }

    /* MOBILE TWEAKS */
    @media (max-width: 600px) {
        .palette {
            grid-template-columns: repeat(auto-fill, minmax(70px, 1fr));
            gap: 10px;
        }

        .color-box {
            height: 90px;
        }

        .controls {
            flex-direction: column;
            align-items: flex-start;
        }

        .controls label {
            min-width: auto;
        }

        body {
            margin: 12px;
        }
    }
</style>
</head>

<body>
<h1>Colour Palettes</h1>
<div id="palette-root"></div>

<script>
// GLOBAL COLOURS
const BRAND = {
  CORP: [
    { name:'Primary Blue', value:'#1b4ba7' }, { name:'Dark Blue', value:'#07142c' }, { name:'Black', value:'#02020a' }, { name:'Light Grey', value:'#f9f9f9' }
  ],
  NEUTRAL: [
    {name:'Silver', value:'#CCCCCC'}, {name:'Steel', value:'#BBBBBB'}, {name:'Ash', value:'#9C9C9C'}
  ]
};
const DATA_COLOURS = {
  contrasting: [
    {name:'Ocean',value:'#0077BB'}, {name:'Coral',value:'#FF8866'}, {name:'Sky',value:'#77CCFF'}, {name:'Pine',value:'#009988'},
    {name:'Gold',value:'#FFDD33'}, {name:'Rust',value:'#AA4400'}, {name:'Lavender',value:'#AA99EE'}, {name:'Silver',value:'#CCCCCC'}
  ],
  rainbow: [
    {name:'Ocean',value:'#0077BB'}, {name:'Lavender',value:'#AA99EE'}, {name:'Sky',value:'#77CCFF'}, {name:'Pine',value:'#009988'},
    {name:'Gold',value:'#FFDD33'}, {name:'Coral',value:'#FF8866'}, {name:'Rust',value:'#AA4400'}, {name:'Silver',value:'#CCCCCC'}
  ]
};
const RAMPS = {
  Ocean:["#004770","#005F96","#0077BB","#66ADD6","#99C9E4","#CCE4F1","#E6F1F8"],
  Coral:["#99523D","#CC6D52","#FF8866","#FFB8A3","#FFCFC2","#FFE7E0","#FFF3F0"],
  Sky:["#477A99","#5FA3CC","#77CCFF","#92D6FF","#ADE0FF","#C9EBFF","#E4F5FF"],
  Pine:["#003D36","#005C52","#009988","#66C2B8","#99D6CF","#CCEBE7","#E6F5F3"],
  Gold:["#A38800","#E0BB00","#FFDD33","#FFEB85","#FFF1AD","#FFF8D6","#FFFCEB"],
  Rust:["#441B00","#662900","#AA4400","#BB6933","#CC8F66","#DDB499","#EEDACC"],
  Lavender:["#665C8F","#887ABE","#AA99EE","#CCC2F5","#DDD6F8","#EEEBFC","#F7F5FD"]
};
const PRIORITY = [2,6,4,1,5,3,0];

// STATE
let singleColour='Ocean', singleSteps=7;
let divergingLeft='Gold', divergingRight='Ocean', divergingSteps=7;

// UTILS
const clamp = (n,min,max)=>Math.max(min,Math.min(max,n));

function maxOddSteps(){
  const minLen = Math.min(...Object.values(RAMPS).map(r=>r.length));
  return 2 * Math.min(minLen, PRIORITY.length) + 1;
}

// RAMP GENERATORS
function generateSingleRamp(name, steps){
  const arr = RAMPS[name];
  const n = clamp(steps,1,arr.length);
  return PRIORITY.slice(0,n).sort((a,b)=>a-b)
    .map((i,k)=>({name:`${name} ${k+1}`, value:arr[i]}));
}

function generateDivergingRamp(left,right,steps){
  let n = steps % 2 ? steps : steps - 1;
  n = clamp(n,3,maxOddSteps());

  const perSide = (n - 1) / 2;
  const idx = PRIORITY.slice(0,perSide).sort((a,b)=>a-b);

  const L = RAMPS[left], R = RAMPS[right];

  const leftRamp  = idx.map((i,k)=>({name:`${left} ${k+1}`, value:L[i]}));
  const rightRamp = idx.map((i,k)=>({name:`${right} ${k+1}`, value:R[i]})).reverse();

  return [...leftRamp, {name:'Neutral',value:'#F9F9F9'}, ...rightRamp];
}

// UI HELPERS
function createEl(tag, props={}, children=[]){
  const el=document.createElement(tag);
  Object.assign(el, props);
  children.forEach(c => el.appendChild(c));
  return el;
}

function createColorBox(c){
  const box=createEl('button',{className:'color-box'});
  box.style.setProperty('--box-color',c.value);

  const caption=createEl('div',{className:'caption'});
  caption.innerHTML=`<strong>${c.name}</strong>${c.value}`;

  const toast=createEl('div',{className:'copied',textContent:'Copied!'});

  box.onclick=()=>{
    navigator.clipboard.writeText(c.value).then(()=>{
      toast.classList.add('visible');
      setTimeout(()=>toast.classList.remove('visible'),900);
    });
  };

  box.append(caption, toast);
  return box;
}

function renderPaletteGroup(parent, group){
  const article=createEl('article',{className:'palette-group'});
  const h3=createEl('h3',{textContent:group.name});
  const grid=createEl('div',{className:'palette'});

  group.colors.forEach(c => grid.appendChild(createColorBox(c)));
  article.append(h3, grid);
  parent.appendChild(article);
}

// CONTROLS
function createSelect(values, current, onChange){
  const sel=createEl('select',{className:'steps'});
  values.forEach(v=>{
    const opt=createEl('option',{value:v,textContent:v});
    if(v===current) opt.selected=true;
    sel.appendChild(opt);
  });
  sel.onchange=e=>onChange(e.target.value);
  return sel;
}

function renderSingleControls(section){
  const ctrl=createEl('div',{className:'controls'});

  ctrl.append(
    createEl('label',{textContent:'Colour:'}),
    createSelect(Object.keys(RAMPS), singleColour, v=>{
      singleColour=v; render();
    }),
    createEl('label',{textContent:'Steps:'}),
    createSelect([2,3,4,5,6,7], singleSteps, v=>{
      singleSteps=parseInt(v); render();
    })
  );

  section.appendChild(ctrl);
}

function renderDivergingControls(section){
  const names = Object.keys(RAMPS);
  const oddSteps = [];
  for(let i=3; i<=maxOddSteps(); i+=2) oddSteps.push(i);

  const ctrl=createEl('div',{className:'controls'});
  ctrl.append(
    createEl('label',{textContent:'Colour 1:'}),
    createSelect(names, divergingLeft, v=>{divergingLeft=v; render();}),
    createEl('label',{textContent:'Colour 2:'}),
    createSelect(names, divergingRight, v=>{divergingRight=v; render();}),
    createEl('label',{textContent:'Steps:'}),
    createSelect(oddSteps, divergingSteps, v=>{
      divergingSteps=parseInt(v); render();
    })
  );

  section.appendChild(ctrl);
}

// SECTION RENDER
function buildSections(){
  return [
    {
      title:'Brand Colours',
      groups:[
        {name:'Corporate brand',colors:BRAND.CORP},
        {name:'Neutral shades', colors:BRAND.NEUTRAL}
      ]
    },
    {
      title:'Data Colours',
      groups:[
        {name:'Contrasting order',colors:DATA_COLOURS.contrasting},
        {name:'Rainbow order',colors:DATA_COLOURS.rainbow}
      ]
    },
    {
      title:'Single Colour Ramps',
      controls:'single',
      groups:[{
        name:`${singleColour} ${singleSteps} steps`,
        colors:generateSingleRamp(singleColour,singleSteps)
      }]
    },
    {
      title:'Diverging Ramps',
      controls:'div',
      groups:[{
        name:`${divergingLeft} – ${divergingRight}`,
        colors:generateDivergingRamp(divergingLeft,divergingRight,divergingSteps)
      }]
    }
  ];
}

function render(){
  const root=document.getElementById('palette-root');
  root.innerHTML='';

  buildSections().forEach(sec=>{
    const section=createEl('section',{className:'palette-section'});
    section.appendChild(createEl('h2',{textContent:sec.title}));

    if(sec.controls==='single') renderSingleControls(section);
    if(sec.controls==='div') renderDivergingControls(section);

    sec.groups.forEach(g => renderPaletteGroup(section,g));
    root.appendChild(section);
  });
}

render();
</script>

</body>