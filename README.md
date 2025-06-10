<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>WebGL Spiral</title>
  <style>
    body { margin: 0; background: black; }
    canvas { display: block; width: 100vw; height: 100vh; }
  </style>
</head>
<body>
<canvas id="glcanvas"></canvas>
<script>
const canvas = document.getElementById('glcanvas');
const gl = canvas.getContext('webgl');

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

// 頂点シェーダー
const vertexShaderSource = `
  attribute float aIndex;
  uniform float uTime;
  uniform float uPointCount;
  uniform vec2 uResolution;
  void main() {
    float i = aIndex;
    float t = uTime * 0.5;
    float angle = i * 0.15 + t;
    float radius = i * 0.003;
    vec2 pos = vec2(cos(angle), sin(angle)) * radius;
    vec2 clipPos = (pos / (uResolution / min(uResolution.x, uResolution.y))) * 2.0;
    gl_Position = vec4(clipPos, 0, 1);
    gl_PointSize = 2.0 + (sin(i * 0.05 + uTime * 2.0) + 1.0) * 2.0;
  }
`;

// フラグメントシェーダー
const fragmentShaderSource = `
  precision mediump float;
  void main() {
    float dist = length(gl_PointCoord - vec2(0.5));
    if (dist > 0.5) discard;
    gl_FragColor = vec4(0.0, 1.0, 0.5, 1.0); // 緑系カラー
  }
`;

// シェーダー作成
function createShader(gl, type, source) {
  const shader = gl.createShader(type);
  gl.shaderSource(shader, source);
  gl.compileShader(shader);
  return shader;
}

// プログラム作成
const vertexShader = createShader(gl, gl.VERTEX_SHADER, vertexShaderSource);
const fragmentShader = createShader(gl, gl.FRAGMENT_SHADER, fragmentShaderSource);
const program = gl.createProgram();
gl.attachShader(program, vertexShader);
gl.attachShader(program, fragmentShader);
gl.linkProgram(program);
gl.useProgram(program);

// 属性とユニフォーム
const aIndex = gl.getAttribLocation(program, 'aIndex');
const uTime = gl.getUniformLocation(program, 'uTime');
const uPointCount = gl.getUniformLocation(program, 'uPointCount');
const uResolution = gl.getUniformLocation(program, 'uResolution');

// 頂点バッファ
const POINT_COUNT = 1000;
const indices = new Float32Array(POINT_COUNT);
for (let i = 0; i < POINT_COUNT; i++) indices[i] = i;
const indexBuffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, indexBuffer);
gl.bufferData(gl.ARRAY_BUFFER, indices, gl.STATIC_DRAW);
gl.enableVertexAttribArray(aIndex);
gl.vertexAttribPointer(aIndex, 1, gl.FLOAT, false, 0, 0);

// 描画ループ
function render(time) {
  gl.viewport(0, 0, canvas.width, canvas.height);
  gl.clear(gl.COLOR_BUFFER_BIT);

  gl.uniform1f(uTime, time * 0.001);
  gl.uniform1f(uPointCount, POINT_COUNT);
  gl.uniform2f(uResolution, canvas.width, canvas.height);

  gl.drawArrays(gl.POINTS, 0, POINT_COUNT);
  requestAnimationFrame(render);
}

gl.clearColor(0, 0, 0, 1);
render(0);
</script>
</body>
</html>
