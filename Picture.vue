<template>
	<div class="data-safe flow webgl">
		<div class="center ">
			<div class="flex" v-if="search_show">
				<input
					type="text"
					class="oinput"
					style="flex:1"
					v-model="value"
					@keyup.enter="search"
				/>
				<button class="search_btn default" @click="search">
					搜索
				</button>
			</div>
			<div id="center"></div>
			<!-- loading动画 -->
			<svg
				v-if="loading"
				xmlns="http://www.w3.org/2000/svg"
				xmlns:xlink="http://www.w3.org/1999/xlink"
				width="30px"
				height="30px"
				viewBox="0 0 50 50"
				style="enable-background:new 0 0 50 50;position:absolute;left:50%;top:50%;transform:translate(-50%,-50%)"
				xml:space="preserve"
			>
				<path
					fill="#FF6700"
					d="M43.935,25.145c0-10.318-8.364-18.683-18.683-18.683c-10.318,0-18.683,8.365-18.683,18.683h4.068c0-8.071,6.543-14.615,14.615-14.615c8.072,0,14.615,6.543,14.615,14.615H43.935z"
					transform="rotate(275.098 25 25)"
				>
					<animateTransform
						attributeType="xml"
						attributeName="transform"
						type="rotate"
						from="0 25 25"
						to="360 25 25"
						dur="0.6s"
						repeatCount="indefinite"
					></animateTransform>
				</path>
			</svg>
			<div style="position:absolute;left:0;display:flex;flex-flow:column">
				<button class="btn success btn_left mt0" @click="search_fn">
					搜索:{{ search_show ? "关闭" : "开启" }}
				</button>
				<button
					class="btn default btn_left"
					@click="changeNodeType(1, 'terminal')"
					:class="{ active: node == 'terminal' }"
				>
					终端
				</button>
				<button
					class="btn default btn_left"
					@click="changeNodeType(2, 'interface')"
					:class="{ active: node == 'interface' }"
				>
					接口
				</button>
				<button
					class="btn default btn_left"
					@click="changeNodeType(9, 'employee')"
					:class="{ active: node == 'employee' }"
				>
					员工
				</button>
				<button
					class="btn default btn_left"
					@click="changeNodeType(4, 'database')"
					:class="{ active: node == 'database' }"
				>
					数据
				</button>
				<button
					class="btn default btn_left"
					:class="{ active: node == 'tracesource' }"
					@click="changeNodeType(5, 'tracesource')"
				>
					溯源
				</button>
			</div>
		</div>
		<!-- 模态框 -->
		<div v-if="dialogVisible" class="dialog">
			<div class="dialog_main">
				<header>
					<span class="close" @click="close"></span>
				</header>
				<!-- 内容 -->
				<div class="echarts">
					<!-- 属性 -->
					<div class="echarts_two"></div>
					<!-- 表 -->
					<div class="echarts_one"></div>
				</div>
			</div>
		</div>
	</div>
</template>
<script>
import {
	getWebglNode,
	getWebglSlug,
	getSafeThings,
	geTracesource,
} from "@u/api.js";
import "@/css/picture.scss";
import * as THREE from "three";
import TWEEN from "@tweenjs/tween.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
//电磁风暴
import { GlitchPass } from "three/examples/jsm/postprocessing/GlitchPass.js";
import { EffectComposer } from "three/examples/jsm/postprocessing/EffectComposer.js";
export default {
	name: "Picture",
	data() {
		return {
			data: [],
			max: 160,
			nodeType: 1,
			node: "terminal",
			value: "",
			first: true,
			speed: 0.0008,
			stop: false,
			search_show: false,
			dialogVisible: false,
			one: "",
			two: "",
			leftEcharts_two: "",
			links_two: "",
			leftEcharts: [],
			echartsName: "",
			links: [],
			echarts_one_data: [],
			echarts_one_links: [],
			loading: true,
			isclicknum: 0,
			twoNodedata: "",
			echarts_two_data: [],
			echarts_two_links: [],
			active: 1,
			concernA: "",
			concernB: "",
			glitchPass: null,
			composer: null,
			clock: null,
			kg: true,
			seach_stop: false,
		};
	},
	mounted() {
		setTimeout(() => {
			this.kg = false;
		}, 1000);
		this.getTable(1, "terminal");
	},
	methods: {
		close() {
			this.active = 1;
			this.dialogVisible = false;
		},
		search_fn() {
			this.search_show = !this.search_show;
			this.seach_stop = false;
			if (!this.search_show) {
				this.value = "";
				this.scene.traverse((obj, i) => {
					if (obj.name) {
						obj.scale.set && obj.scale.set(80, 40, 1);
						obj.material && obj.material.color.set(0x5dc1da);
					}
				});
			}
		},
		search() {
			if (!this.value) {
				return this.$message.warning("搜索数据不能为空！");
			} else if (
				this.data.every((v) => {
					return v.name != this.value;
				})
			) {
				return this.$message.warning("没找到当前数据！");
			}
			this.seach_stop = true;
			let arr = [];
			this.scene.traverse((obj, i) => {
				if (obj.name) {
					obj.material && obj.material.color.set(0x5dc1da);
					obj.scale.set && obj.scale.set(80, 40, 1);
				}
			});
			this.scene.traverse((obj, i) => {
				if (obj.name) {
					if (obj.name.name == this.value) {
						arr.push(obj.position);
						obj.material && obj.material.color.set(0xffdd00);
						obj.scale.set && obj.scale.set(320, 160, 1);
					}
				}
			});
			if (arr.length <= 0) {
				arr = [];
				this.seach_stop = false;
				return this.$message.warning("没找到当前数据！");
			}
			if (this.node == "tracesource") {
				arr = [];
				return (this.seach_stop = true);
			}
			let { x, y, z } = arr[0];
			this.group.rotation.set(0, 0, 0);
			let This = this;
			let tween = new TWEEN.Tween({
				px: this.camera.position.x,
				py: this.camera.position.y,
				pz: this.camera.position.z,
			});
			tween.to(
				{
					px: x,
					py: y,
					pz: z,
				},
				1000
			);
			tween.onUpdate(function(param) {
				This.camera.position.set(param.px, param.py, param.pz);
				This.camera.lookAt(0, 0, 0);
				This.camera.translateZ(1000);
			});
			tween.easing(TWEEN.Easing.Cubic.InOut);
			tween.start();
		},
		changeNodeType(index, node) {
			this.nodeType = index;
			this.node = node;
			this.value = "";
			if (node == "tracesource" && index == 5) {
				this.loading = true;
				getSafeThings({}).then((res) => {
					if (res.results && res.results.length >= 160) {
						this.data = res.results
							? res.results.map((v) => {
									v.color = "#5dc1da";
									v.slug = v.rawDatabaseFlow
										? v.rawDatabaseFlow
										: v.rawProtocolFlow;
									return v;
							  })
							: [];
					} else {
						let odata = res.results
							? res.results.map((v) => {
									v.color = "#5dc1da";
									v.slug = v.rawDatabaseFlow
										? v.rawDatabaseFlow
										: v.rawProtocolFlow;
									return v;
							  })
							: [];
						this.data = this.addData(odata);
					}
					this.max = this.data.length;
					this.loading = false;
					this.seach_stop = false;
					this.init();
				});
			} else {
				this.getTable(this.nodeType, node);
			}
		},
		addData(data) {
			if (data.length >= 160) {
				return data;
			} else {
				return this.addData(data.concat(data));
			}
		},
		getTable(nodeType, node) {
			let nodename = node;
			this.loading = true;
			getWebglNode({ nodeType }).then((res) => {
				if (res.result && res.result[nodename].length >= 160) {
					this.data = res.result
						? res.result[nodename].map((v) => {
								v.color = "#5dc1da";
								return v;
						  })
						: [];
				} else {
					let odata = res.result
						? res.result[nodename].map((v) => {
								v.color = "#5dc1da";
								return v;
						  })
						: [];

					this.data = this.addData(odata);
				}
				this.max = this.data.length;
				this.loading = false;
				this.seach_stop = false;
				this.init();
			});
		},
		makeTextCanvas(text, width, height) {
			this.canvas = document.createElement("canvas");
			this.canvas.width = width;
			this.canvas.height = height;
			var c = this.canvas.getContext("2d");
			c.clearRect(0, 0, c.canvas.width, c.canvas.height);
			c.beginPath();
			c.translate(width / 2, height / 2);
			c.fillStyle = "#5dc1da";
			c.font = "48px Arial";
			c.textBaseline = "middle";
			c.textAlign = "center";
			c.fillText(text.name, 0, 0);
			return c.canvas;
		},
		createS(i) {
			var textCanvas = this.makeTextCanvas(
				this.data[i] ? this.data[i] : "null",
				320,
				80
			);
			var texture = new THREE.CanvasTexture(textCanvas);
			texture.generateMipmaps = false;
			texture.encoding = THREE.sRGBEncoding;
			texture.minFilter = THREE.LinearFilter;
			texture.magFilter = THREE.LinearFilter;

			let pinMaterial = new THREE.SpriteMaterial({
				map: texture,
			});
			let mesh = new THREE.Sprite(pinMaterial);
			mesh.name = {
				slug: this.data[i].slug ? this.data[i].slug : null,
				name: this.data[i].name ? this.data[i].name : null,
				data: this.data[i] ? this.data[i] : null,
			};
			return mesh;
		},
		render() {
			this.renderer.render(this.scene, this.camera);
			if (!this.stop && !this.seach_stop) {
				this.group.rotateY(-0.0008);
				this.group.rotateX(0.0008);
			}
			//电磁风暴
			// this.createComposer();
			// this.controls.update();
			// this.composer.render(delta);
			TWEEN.update();
			requestAnimationFrame(this.render);
		},
		onResize(v) {
			setTimeout(() => {
				this.width = document.querySelector("#center").offsetWidth;
				this.height = document.querySelector("#center").offsetHeight;
				this.renderer.setSize(this.width, this.height);
				if (this.camera) {
					this.camera.aspect = this.width / this.height;
					this.camera.updateProjectionMatrix();
				}
				if (this.one) {
					this.one.clear();
					this.node == "tracesource" ? this.drawThree() : this.drawOne();
				}
			});
		},
		createComposer() {
			this.glitchPass = new GlitchPass(64);
			this.glitchPass.enabled = true;
			this.composer = new EffectComposer(this.renderer);
			this.composer.addPass(this.glitchPass);
			this.composer.passes[0].enabled = this.kg;
		},
		onMouseClick(event) {
			event.preventDefault();
			let dom = document.querySelector("#center");
			this.mouse.x =
				((event.clientX - dom.getBoundingClientRect().left) / dom.offsetWidth) *
					2 -
				1;
			this.mouse.y =
				-(
					(event.clientY - dom.getBoundingClientRect().top) /
					dom.offsetHeight
				) *
					2 +
				1;
			this.raycaster.setFromCamera(this.mouse, this.camera);
			this.intersects = this.raycaster.intersectObjects(
				this.scene.children[1].children
			);
			if (this.intersects.length > 0 && !this.dialogVisible) {
				if (
					this.intersects[0].object &&
					this.intersects[0].object.name &&
					this.intersects[0].object.name.slug
				) {
					if (this.node == "tracesource") {
						geTracesource({
							rawDatabaseFlow: this.intersects[0].object.name.data
								.rawDatabaseFlow
								? this.intersects[0].object.name.data.rawDatabaseFlow.id
								: null,
							rawProtocolFlow: this.intersects[0].object.name.data
								.rawProtocolFlow
								? this.intersects[0].object.name.data.rawProtocolFlow.id
								: null,
						}).then((res) => {
							this.dialogVisible = true;
							let attr = this.intersects[0].object.name.data;
							let name = this.intersects[0].object.name.name;
							let data_attr = [];
							let links_attr = [];
							let links = [];
							let data = [];
							let keys = {
								Company: "部门",
								Department: "员工",
								Employee: "拥有",
								Ip: "访问",
								Url: "",
								Dest_ip: "",
							};
							let attr_length = Object.keys(attr).length;
							data_attr[0] = { name: name };
							Object.keys(attr).forEach((v, i) => {
								if (v != "color" && v != "slug") {
									data_attr.push({
										name:
											Object.prototype.toString.call(attr[v]) ==
											"[object Object]"
												? "object"
												: attr[v],
									});
									if (i < attr_length - 1) {
										links_attr.push({
											source: 0,
											target: i + 1,
											value: v,
										});
									}
								}
							});
							data_attr = data_attr.map((v, i) => {
								v.id = i;
								return v;
							});
							this.leftEcharts_two = data_attr;
							this.links_two = links_attr;
							this.drawTwo();
							Object.keys(res).forEach((v, i) => {
								data.push({
									key: v,
									name: res[v],
									slug: null,
									value: [i * (i * 3 + 15), 5],
									btn: 4,
								});
							});
							data.slice(0).forEach((v, i) => {
								if (i + 1 < data.length) {
									links.push({
										source: data[i].name,
										target: data[i + 1].name,
										value: keys[v.key] ? keys[v.key] : "",
									});
								}
							});
							this.leftEcharts = data;
							this.links = links;
							this.drawThree();
						});
					} else {
						getWebglSlug({ slug: this.intersects[0].object.name.slug }).then(
							(res) => {
								this.dialogVisible = true;
								let attr = this.intersects[0].object.name.data;
								let name = this.intersects[0].object.name.name;
								let data_attr = [];
								let links_attr = [];
								let links = [];
								let attr_length = Object.keys(attr).length;
								data_attr[0] = { name: name };
								Object.keys(attr).forEach((v, i) => {
									if (v != "color") {
										data_attr.push({
											name: attr[v],
										});
										if (i < attr_length - 1) {
											links_attr.push({
												source: 0,
												target: i + 1,
												value: v,
											});
										}
									}
								});
								data_attr = data_attr.map((v, i) => {
									v.id = i;
									return v;
								});
								this.leftEcharts_two = data_attr;
								this.links_two = links_attr;
								this.drawTwo();
								let terminal = res.groupResult.terminal;
								let data = [];
								let table_length;
								let value;
								let concernA = "";
								let concernB = "";
								let variables = "name";
								if (this.node == "terminal") {
									table_length = res.groupResult.database.length;
									value = "database";
									concernA = "库";
									concernB = "";
									variables = "name";
								} else if (this.node == "database") {
									table_length = res.groupResult.table.length;
									value = "table";
									concernA = "表";
									concernB = "库";
									variables = "name";
								} else if (this.node == "employee") {
									table_length = res.groupResult.department.length;
									value = "department";
									concernA = "员工";
									concernB = "";
									variables = "name";
								} else if (this.node == "interface") {
									table_length = res.groupResult.ip.length;
									value = "ip";
									concernA = "访问";
									concernB = "";
									variables = "ip";
								}
								data[0] = {
									name: name,
									btn: 1,
									symbolSize: 40,
									itemStyle: {
										normal: {
											borderColor: "#b457ff",
											borderWidth: 4,
											shadowBlur: 10,
											shadowColor: "#b457ff",
											color: "#001c43",
										},
									},
								};
								if (terminal.length <= 0) {
									res.groupResult[value].forEach((v) => {
										data.push({
											name: v[variables],
											slug: v.slug,
											btn: 2,
											symbolSize: 35,
											itemStyle: {
												normal: {
													borderColor: "#04f2a7",
													borderWidth: 4,
													shadowBlur: 10,
													shadowColor: "#04f2a7",
													color: "#ff579a",
												},
											},
										});
									});
									if (this.node == "terminal") {
										res.groupResult.field.forEach((v) => {
											data.push({
												name: v[variables],
												slug: v.slug,
												btn: 2,
												symbolSize: 35,
												itemStyle: {
													normal: {
														borderColor: "#04f2a7",
														borderWidth: 4,
														shadowBlur: 10,
														shadowColor: "#04f2a7",
														color: "#ff579a",
													},
												},
											});
										});
									}
									if (this.node != "interface") {
										links = data.slice(1).map((v, i) => {
											return {
												source: 0,
												target: i + 1,
												value: concernA,
											};
										});
									} else {
										links = data.slice(1).map((v, i) => {
											return {
												source: i + 1,
												target: 0,
												value: concernA,
											};
										});
									}
									this.echarts_one_links = links;
									this.links = links;
								} else {
									res.groupResult[value].forEach((v) => {
										data.push({
											symbolSize: 35,
											name: v[variables],
											slug: v.slug,
											btn: 2,
											itemStyle: {
												normal: {
													borderColor: "#04f2a7",
													borderWidth: 4,
													shadowBlur: 10,
													shadowColor: "#04f2a7",
													color: "#ff579a",
												},
											},
										});
									});

									terminal.forEach((v) => {
										data.push({
											name: v[variables],
											slug: v.slug,
											btn: 2,
											symbolSize: 35,
											itemStyle: {
												normal: {
													borderColor: "#04f2a7",
													borderWidth: 4,
													shadowBlur: 10,
													shadowColor: "#04f2a7",
													color: "#ff579a",
												},
											},
										});
									});
									let testa = data.slice(1, table_length + 1).map((v, i) => {
										return {
											source: 0,
											target: i + 1,
											value: concernA,
										};
									});
									let testb = data.slice(table_length + 1).map((v, i) => {
										return {
											source: table_length + i + 1,
											target: 0,
											value: concernB,
										};
									});
									this.links = testa.concat(testb);
									this.echarts_one_links = this.links;
								}
								data = data.map((v, i) => {
									v.id = i;
									return v;
								});
								this.concernA = concernA;
								this.concernB = concernB;
								this.echarts_one_data = data;
								this.leftEcharts = data;
								this.drawOne();
							}
						);
					}
				} else if (!this.intersects[0].object.name.slug) {
					this.$message.error("slug为空");
				}
			}
		},
		init() {
			document.querySelector("#center").innerHTML = "";
			this.width = document.querySelector("#center").offsetWidth;
			this.height = document.querySelector("#center").offsetHeight;
			this.scene = new THREE.Scene();
			this.point = new THREE.PointLight(0xffffff);
			if (this.first) {
				this.point.position.set(400, 200, 300);
			}
			var ambient = new THREE.AmbientLight(0x444444);
			this.scene.add(ambient);
			this.camera = new THREE.PerspectiveCamera(
				35,
				this.width / this.height,
				1,
				10000
			);
			this.raycaster = new THREE.Raycaster();
			this.mouse = new THREE.Vector2();
			this.camera.position.set(0, 0, 3000);
			this.group = new THREE.Group();
			for (var i = 0, l = this.max; i < l; i++) {
				let phi = Math.acos(-1 + (2 * i) / l);
				let theta = Math.sqrt(l * Math.PI) * phi;
				let sprite = this.createS(i);
				sprite.scale.set(80, 40, 1);
				sprite.position.setFromSphericalCoords(800, phi, theta);
				this.group.add(sprite);
			}
			this.scene.add(this.group);
			this.renderer = new THREE.WebGLRenderer({
				antialias: true,
				alpha: true,
			});
			this.renderer.setSize(this.width, this.height);

			this.renderer.shadowMap.enabled = false;
			this.renderer.setClearColor(0xeeeeee, 0.0);
			this.renderer.setClearAlpha(0.0);
			this.renderer.setPixelRatio(window.devicePixelRatio);
			document.querySelector("#center").appendChild(this.renderer.domElement);
			this.renderer.gammaOutput = true;
			this.renderer.gammaFactor = 2.2;

			this.renderer.render(this.scene, this.camera);
			this.clock = new THREE.Clock();
			this.controls = new OrbitControls(this.camera, this.renderer.domElement);
			if (this.first) {
				this.render();
				this.first = false;
			}
			document
				.querySelector("canvas")
				.addEventListener("click", this.onMouseClick, false);
			window.addEventListener("resize", this.onResize, false);
			document.querySelector("canvas").addEventListener("mouseenter", () => {
				this.stop = true;
			});
			document.querySelector("canvas").addEventListener("mouseleave", () => {
				this.stop = false;
			});
		},
		drawOne() {
			this.$nextTick(() => {
				this.one = this.echarts.init(document.querySelector(".echarts_one"));
				this.one.off("click");
				this.one.setOption({
					backgroundColor: "#0a202b",
					grid: {
						left: "10%",
						top: 60,
						right: "10%",
						bottom: 60,
					},
					animationDuration: 1500,
					animationDurationUpdate: 1500,
					color: ["#83e0ff", "#45f5ce", "#b158ff"],
					xAxis: {
						show: false,
						type: "value",
						boundaryGap: false,
						min: 0,
						max: 120,
					},
					yAxis: {
						show: false,
						type: "value",
						min: -10,
						max: 20,
					},
					series: [
						{
							type: "graph",
							layout: "force",
							force: {
								repulsion: 800,
								edgeLength: 10,
								layoutAnimation: false,
							},
							symbolSize: 30,
							nodeScaleRatio: 1,
							roam: true,
							draggable: false,
							focusNodeAdjacency: false,
							edgeSymbol: ["circle", "arrow"],
							label: {
								normal: {
									show: true,
									position: "inside",
									color: "gold",
								},
							},
							edgeLabel: {
								normal: {
									show: true,
									textStyle: {
										fontSize: 12,
										color: "#fff",
									},
									formatter: "{c}",
								},
							},
							itemStyle: {
								normal: {
									borderColor: "#04f2a7",
									borderWidth: 2,
									shadowBlur: 10,
									shadowColor: "#04f2a7",
									color: "#001c43",
								},
							},
							lineStyle: {
								normal: {
									opacity: 0.9,
									width: 2,
									curveness: 0,
									color: {
										type: "linear",
										x: 0,
										y: 0,
										x2: 0,
										y2: 1,
										colorStops: [
											{
												offset: 0,
												color: "#e0f55a",
											},
											{
												offset: 1,
												color: "#639564",
											},
										],
										globalCoord: false,
									},
								},
							},
							symbolKeepAspect: true,
							data: this.leftEcharts,
							links: this.links,
						},
					],
				});
				this.one.on("click", (params) => {
					this.throttle(this.change(params.data), 500);
				});
			});
		},
		change(item) {
			if (
				this.echarts_one_data[0] &&
				item.name == this.echarts_one_data[0].name &&
				item.btn == 1 &&
				item.btn != this.active
			) {
				this.active = 1;
				this.leftEcharts = this.echarts_one_data;
				this.links = this.echarts_one_links;
				this.setOption();
			} else if (item.btn == 2 && item.btn != this.active) {
				if (this.node == "terminal" || this.node == "interface") {
					return;
				}
				this.active = 2;
				this.throttle(
					getWebglSlug({ slug: item.slug }).then((res) => {
						this.twoNodedata = {
							name: item.name,
							symbolSize: 30,
							slug: item.slug,
							itemStyle: {
								normal: {
									borderColor: "#04f2a7",
									borderWidth: 4,
									shadowBlur: 10,
									shadowColor: "#04f2a7",
									color: "#ff579a",
								},
							},
							btn: 2,
						};
						let data = [
							{
								name: this.echarts_one_data[0].name,
								symbolSize: 40,
								btn: 1,
								itemStyle: {
									normal: {
										borderColor: "#b457ff",
										borderWidth: 4,
										shadowBlur: 10,
										shadowColor: "#b457ff",
										color: "#001c43",
									},
								},
							},
							{
								name: item.name,
								symbolSize: 35,
								slug: item.slug,
								itemStyle: {
									normal: {
										borderColor: "#04f2a7",
										borderWidth: 4,
										shadowBlur: 10,
										shadowColor: "#04f2a7",
										color: "#ff579a",
									},
								},
								btn: 2,
							},
						];
						let link = [];
						if (this.node == "database") {
							res.groupResult.field.map((v) => {
								data.push({
									name: v.name,
									btn: 3,
									slug: v.slug,
									symbolSize: 30,
								});
							});
							link = [
								{
									source: 0,
									target: 1,
									value: "表",
								},
							];
							data.slice(1).map((v, i) => {
								if (i != 0) {
									link.push({
										source: 1,
										target: i + 1,
										value: "字段",
									});
								}
							});
						} else if (this.node == "employee") {
							res.groupResult.company.map((v) => {
								data.push({
									name: v.name,
									btn: 3,
									slug: v.slug,
									symbolSize: 30,
								});
							});
							link = [
								{
									source: 0,
									target: 1,
									value: "员工",
								},
							];
							data.slice(1).map((v, i) => {
								if (i != 0) {
									link.push({
										source: i + 1,
										target: 1,
										value: "公司",
									});
								}
							});
						}
						this.leftEcharts = data;
						this.links = link;
						this.setOption();
					}, 100),
					100
				);
			} else if (
				item.btn == 3 &&
				item.btn != this.active &&
				this.node == "database"
			) {
				this.active = 3;
				let oslug = item.slug;
				this.throttle(
					getWebglSlug({ slug: oslug }).then((res) => {
						let data = [
							{
								name: this.echarts_one_data[0].name,
								symbolSize: 40,
								btn: 1,
								itemStyle: {
									normal: {
										borderColor: "#b457ff",
										borderWidth: 4,
										shadowBlur: 10,
										shadowColor: "#b457ff",
										color: "#001c43",
									},
								},
							},
							this.twoNodedata,
							{ name: item.name, symbolSize: 30, btn: 3, slug: item.slug },
						];
						res.groupResult.terminal.map((v) => {
							data.push({ name: v.name, btn: 4, slug: v.slug, symbolSize: 15 });
						});
						let link = [
							{
								source: 0,
								target: 1,
								value: "表",
							},
							{
								source: 1,
								target: 2,
								value: "字段",
							},
						];
						data.slice(2).map((v, i) => {
							if (i != 0) {
								link.push({
									source: 2 + i,
									target: 2,
									value: "访问",
								});
							}
						});
						data = data.map((v, i) => {
							v.id = i;
							return v;
						});
						this.leftEcharts = data;
						this.links = link;
						this.setOption();
					}, 100),
					100
				);
			}
		},
		setOption() {
			setTimeout(() => {
				this.one.clear();
				this.drawOne();
			});
		},
		throttle(fn, wait) {
			var pre = Date.now();
			return function() {
				var context = this;
				var args = arguments;
				var now = Date.now();
				if (now - pre >= wait) {
					fn.apply(context, args);
					pre = Date.now();
				}
			};
		},
		drawThree() {
			this.$nextTick(() => {
				this.one = this.echarts.init(document.querySelector(".echarts_one"));
				this.one.off("click");
				this.one.setOption({
					backgroundColor: "#0a202b",
					grid: {
						left: "10%",
						top: 60,
						right: "10%",
						bottom: 60,
					},
					animationDuration: 1500,
					animationDurationUpdate: 1500,
					color: ["#83e0ff", "#45f5ce", "#b158ff"],
					xAxis: {
						show: false,
						type: "value",
						boundaryGap: false,
						min: -10,
						max: 110,
					},
					yAxis: {
						show: false,
						type: "value",
						min: -10,
						max: 20,
					},
					series: [
						{
							coordinateSystem: "cartesian2d",
							type: "graph",
							layout: "force",
							force: {
								repulsion: 800,
								edgeLength: 10,
								layoutAnimation: true,
							},
							symbolSize: 50,
							roam: true,
							draggable: true,
							focusNodeAdjacency: false,
							edgeSymbol: ["circle", "arrow"],
							label: {
								normal: {
									show: true,
									position: "inside",
									color: "gold",
								},
							},
							edgeLabel: {
								normal: {
									show: true,
									textStyle: {
										fontSize: 12,
										color: "#fff",
									},
									formatter: "{c}",
								},
							},
							itemStyle: {
								normal: {
									borderColor: "#04f2a7",
									borderWidth: 2,
									shadowBlur: 10,
									shadowColor: "#04f2a7",
									color: "#001c43",
								},
							},
							lineStyle: {
								normal: {
									opacity: 0.9,
									width: 2,
									curveness: 0,
									color: {
										type: "linear",
										x: 0,
										y: 0,
										x2: 0,
										y2: 1,
										colorStops: [
											{
												offset: 0,
												color: "#e0f55a",
											},
											{
												offset: 1,
												color: "#639564",
											},
										],
										globalCoord: false,
									},
								},
							},
							symbolKeepAspect: false,
							data: this.leftEcharts,
							links: this.links,
						},
					],
				});
			});
		},
		drawTwo() {
			this.$nextTick(() => {
				this.two = this.echarts.init(document.querySelector(".echarts_two"));
				this.two.setOption({
					backgroundColor: "#0a202b",
					grid: {
						left: "10%",
						top: 60,
						right: "10%",
						bottom: 60,
					},
					animationDuration: 3000,
					animationDurationUpdate: 3000,
					color: ["#83e0ff", "#45f5ce", "#b158ff"],
					series: [
						{
							type: "graph",
							layout: "force",
							force: {
								repulsion: 800,
								edgeLength: 10,
								layoutAnimation: false,
							},
							symbolSize: 30,
							nodeScaleRatio: 1,
							roam: true,
							draggable: false,
							focusNodeAdjacency: true,
							edgeSymbol: ["circle", "arrow"],
							label: {
								normal: {
									show: true,
									position: "inside",
									color: "gold",
								},
							},
							edgeLabel: {
								normal: {
									show: true,
									textStyle: {
										fontSize: 12,
										color: "#fff",
									},
									formatter: "{c}",
								},
							},
							itemStyle: {
								normal: {
									borderColor: "#04f2a7",
									borderWidth: 2,
									shadowBlur: 10,
									shadowColor: "#04f2a7",
									color: "#001c43",
								},
							},
							lineStyle: {
								normal: {
									opacity: 0.9,
									width: 2,
									curveness: 0,
									color: {
										type: "linear",
										x: 0,
										y: 0,
										x2: 0,
										y2: 1,
										colorStops: [
											{
												offset: 0,
												color: "#e0f55a",
											},
											{
												offset: 1,
												color: "#639564",
											},
										],
										globalCoord: true,
									},
								},
							},
							symbolKeepAspect: false,
							data: this.leftEcharts_two,
							links: this.links_two,
						},
					],
				});
			});
		},
	},
};
</script>
