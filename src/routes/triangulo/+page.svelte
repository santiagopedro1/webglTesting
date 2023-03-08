<script lang="ts">
	import { onMount } from 'svelte'

	function initShaders(gl: WebGLRenderingContext) {
		const vertexShader = gl.createShader(gl.VERTEX_SHADER)
		const fragmentShader = gl.createShader(gl.FRAGMENT_SHADER)

		if (!vertexShader || !fragmentShader) {
			alert('Falha ao criar os shaders')
			return
		}

		gl.shaderSource(
			vertexShader,
			`
      attribute vec4 a_Position;
      void main() {
        gl_Position = a_Position;
        gl_PointSize = 10.0;
      }
    `
		)

		gl.shaderSource(
			fragmentShader,
			`
      precision mediump float;
      uniform vec4 u_FragColor;
      void main() {
        gl_FragColor = u_FragColor;
      }
    `
		)

		gl.compileShader(vertexShader)
		gl.compileShader(fragmentShader)

		const program = gl.createProgram()
		if (!program) {
			alert('Falha ao criar o programa')
			return
		}
		gl.attachShader(program, vertexShader)
		gl.attachShader(program, fragmentShader)
		gl.linkProgram(program)

		gl.useProgram(program)

		return program
	}

	function changeColor() {
		const canvas = document.getElementById('canvas') as HTMLCanvasElement
		const gl = canvas.getContext('webgl') as WebGLRenderingContext

		const program = initShaders(gl) as WebGLProgram

		const u_FragColor = gl.getUniformLocation(program, 'u_FragColor')

		const r = Math.random()
		const g = Math.random()
		const b = Math.random()
		const a = Math.random()

		gl.uniform4f(u_FragColor, r, g, b, a)
		gl.clear(gl.COLOR_BUFFER_BIT)
		gl.drawArrays(gl.TRIANGLES, 0, 3)
	}

	onMount(() => {
		const canvas = document.getElementById('canvas') as HTMLCanvasElement
		const gl = canvas.getContext('webgl')

		if (!gl) {
			alert(
				'Incapaz de inicializar o WebGL.Seu navegador ou sua máquina não suporta.'
			)
			return
		}

		gl.clearColor(0.0, 0.0, 0.0, 1.0)
		gl.clear(gl.COLOR_BUFFER_BIT)

		const program = initShaders(gl)

		if (!program) {
			alert('Falha ao inicializar os shaders')
			return
		}

		const a_Position = gl.getAttribLocation(program, 'a_Position')
		const u_FragColor = gl.getUniformLocation(program, 'u_FragColor')

		if (a_Position < 0 || !u_FragColor) {
			alert('Falha ao obter a localização do atributo ou da variável uniforme')
			return
		}

		const vertices = new Float32Array([0.0, 0.5, -0.5, -0.5, 0.5, -0.5])

		const n = 3

		const vertexBuffer = gl.createBuffer()
		if (!vertexBuffer) {
			alert('Falha ao criar o buffer do vértice')
			return
		}

		gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer)
		gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW)

		gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 0, 0)
		gl.enableVertexAttribArray(a_Position)

		gl.uniform4f(u_FragColor, 1.0, 0.0, 0.0, 1.0)

		gl.drawArrays(gl.TRIANGLES, 0, n)
	})
</script>

<canvas
	id="canvas"
	width="1600"
	height="800"
	class="mx-auto"
/>
<div class="mt-3 flex justify-center">
	<button
		class="rounded-xl bg-teal-500 p-4 text-white"
		on:click={changeColor}
	>
		Change triangle color
	</button>
</div>
