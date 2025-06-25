## Квадраты

```c
#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float rect(in vec2 st, in vec4 size) {
    return 
        step(size[0], st.x) *
        (1. - step(size[1], st.x))*
        step(size[2], st.y) *
        (1. - step(size[3], st.y));
}

void main() {
    vec2 st = gl_FragCoord.xy/u_resolution.xy;
   	vec3 color = vec3(0.820,0.751,0.677);
    
    vec3 c_black = vec3(0.01);
    vec3 c_red = vec3(0.645,0.147,0.178);
    vec3 c_yellow = vec3(0.940,0.720,0.188);
    vec3 c_cream = vec3(0.820,0.751,0.677);
    vec3 c_blue = vec3(0.189,0.424,0.820);
    
    float rect_n1 = rect(st, vec4(0.0, 0.2, 0.6, 1.0));
    float rect_n2 = rect(st, vec4(0.9, 1., 0.6, 1.0));
    float rect_n3 = rect(st, vec4(0.7, 1., 0.0, 0.1));

    float line_v1 = rect(st, vec4(0.08, 0.1, 0.6, 1.));
    float line_v2 = rect(st, vec4(0.2, 0.22, 0.0, 1.));
    float line_v3 = rect(st, vec4(0.7, 0.72, 0.0, 1.));
    float line_v4 = rect(st, vec4(0.9, 0.92, 0.0, 1.));

    float line_h1 = rect(st, vec4(0.0, 1, 0.75, 0.77));
    float line_h2 = rect(st, vec4(0.0, 1, 0.6, 0.62));
    float line_h3 = rect(st, vec4(0.2, 1, 0.1, 0.12));
    
    color = mix(color, c_red, rect_n1);
    color = mix(color, c_yellow, rect_n2);
    color = mix(color, c_blue, rect_n3);
    
    color = mix(color, c_black, line_v1);
    color = mix(color, c_black, line_v2);
    color = mix(color, c_black, line_v3);
    color = mix(color, c_black, line_v4);
    
    color = mix(color, c_black, line_h1);
    color = mix(color, c_black, line_h2);
    color = mix(color, c_black, line_h3);
    
    gl_FragColor = vec4(color, 1.0);
}
```


## Геометрическая фигура с угловым шагом

```c
#ifdef GL_ES
precision mediump float;
#endif

#define PI 3.14159265359
#define TWO_PI 6.28318530718

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

// Reference to
// http://thndl.com/square-shaped-shaders.html

void main(){
  vec2 st = gl_FragCoord.xy/u_resolution.xy;
  st.x *= u_resolution.x/u_resolution.y;
  vec3 color = vec3(0.0);
  float d = 0.0;

  // Remap the space to -1. to 1.
  st = st *2.-1.;

  // Number of sides of your shape
  int N = 3;

  // Угол между текущим пикселем и центром экрана (в радианах).
  float a = atan(st.x,st.y);
  // Угловой шаг
  float r = TWO_PI * 1./float(N);

  // d - вычисляет расстояние от текущего пикселя до ближайшего края фигуры

  // - `a / r`: Делим текущий угол `a` на угловой шаг `r`. Это позволяет определить, к какому сегменту фигуры (например, к какой стороне треугольника) принадлежит текущий пиксель.
    
  // - `0.5 + a / r`: Добавляем 0.50.5, чтобы правильно округлить результат. Это нужно, чтобы границы сегментов были точно посередине между вершинами.
    
 //- `floor(...)`: Округляет результат вниз до ближайшего целого числа. Это определяет индекс ближайшего сегмента.
 
 // floor(0.5 + a / r) * r - Умножаем индекс сегмента на угловой шаг `r`, чтобы получить угол ближайшего края фигуры.

// floor(0.5 + a / r) * r - a - Вычитаем текущий угол `a` из угла ближайшего края. Это дает разницу между текущим положением пикселя и ближайшим краем фигуры.

// cos(floor(0.5 + a / r) * r - a) - - Вычисляем косинус разницы углов. Это нужно для того, чтобы определить, насколько близко текущий пиксель находится к краю фигуры. Косинус достигает максимума (11), когда разница углов равна 00, и уменьшается по мере удаления от края.

// - Умножаем косинус разницы углов на расстояние от центра до пикселя. Это дает итоговое расстояние `d`, которое используется для определения, находится ли пиксель внутри фигуры.
 
  d = cos(floor(.5+a/r)*r-a)*length(st);

  color = vec3(1.0-smoothstep(.4,.41,d));

  gl_FragColor = vec4(color,1.0);
}

```

## [Прицел как у ЖеЛеЗнОгО ЧеЛоВеКа](https://thebookofshaders.com/08/?lan=ru)


```c
// Author:
// Title:

#ifdef GL_ES
precision mediump float;
#endif

#define PI 3.14159265359
#define TWO_PI 6.28318530718

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float circlePlot(float dist, in float size, float weight) {
    return smoothstep(size, size + 0.01, dist) * 
        (1. - smoothstep(size + weight, size + weight + 0.005, dist));
}

float circleGrad(float dist, float size, float weight) {
    return smoothstep(size - weight, size + 0.03, dist) * (1. - smoothstep(size, size + 0.01, dist));
}

float circleFull(float dist, float size) {
    return 1. - smoothstep(size, size + 0.01, dist);
}

mat2 rotate2d(float _angle) {
    return mat2(
		 cos(_angle), -sin(_angle), 
		 sin(_angle), cos(_angle)
	 );
}

mat2 scale(vec2 _scale) {
    return mat2(
		_scale.x, 0.0,
		0.0, _scale.y
	); 
}

float line(float dist, vec2 center, float weight) {
    return smoothstep(-1. * weight - 0.00, -1. * weight, dist) * 
        (1. - smoothstep(weight, weight + 0.002, dist));
}

// Тут чисто на костылях
float triangle(vec2 st, vec2 center, float rotate) {
    int N = 3;
    vec2 _st = (st - center) * scale(vec2(1.5, 1.)) * rotate2d(rotate);
    float a = atan(_st.x, _st.y);
    float r = TWO_PI / float(N);
    float d = cos(floor(0.5 + a / r) * r - a) * length(_st);
    return 1. - smoothstep(0.011, 0.012, d);
}

void main() {
    vec2 st = gl_FragCoord.xy / u_resolution.xy;
    
    vec3 CLR_BLUE = vec3(0.410,1.000,0.990);
    vec3 CLR_ORANGE = vec3(1.000,0.380,0.197);
    
    st = st * 2. - 1.;
    
    vec2 center = vec2(0., 0.);
    vec2 targCenter = vec2(0., 0.27);
    
    vec3 color = vec3(0.);
    
    vec3 cplot_1 = vec3(circlePlot(length(st - center), 0.03, 0.008));
    vec3 cplot_2 = vec3(circlePlot(length(st - center), 0.28, 0.008));
	vec3 cplot_3 = vec3(circlePlot(length(st - center), 0.46, 0.008));
    vec3 cplot_4 = vec3(circlePlot(length(st - center), 0.64, 0.008));
    vec3 cplot_5 = vec3(circlePlot(length(st - center), 0.71, 0.008));
    vec3 cplot_6 = vec3(circlePlot(length(st - center), 0.85, 0.012));
    
    vec3 ctarg_1 = vec3(circleFull(length(st - targCenter), 0.02));
    vec3 ctarg_2 = vec3(circlePlot(length(st - targCenter), 0.025, 0.012));
    vec3 ctarg_3 = vec3(circleGrad(length(st - targCenter), 0.24, 0.07));
    
    vec3 cfull_1 = vec3(circleFull(length(st - vec2(0.05, 0.17)), 0.005));    
    vec3 cfull_2 = vec3(circleFull(length(st - vec2(-0.2, 0.55)), 0.005));
    
    
    vec3 line_1 = vec3(line(st.x + st.y, center, 0.001)) * circleFull(length(st - center), 0.64);
    vec3 line_2 = vec3(line(st.x - st.y, center, 0.001)) * circleFull(length(st - center), 0.64);
	vec3 line_3 = vec3(line(st.y, center, 0.002)) * circleFull(length(st - vec2(-0.32, 00)), 0.32);
    

	
    vec3 triangle_1 = vec3(triangle(st, vec2(-0.8, 0.), 34.));
    vec3 triangle_2 = vec3(triangle(st, vec2(0.8, 0.), -34.));
    
    // вот тут вообще хз
    vec3 grad = vec3(st.y + 0.9) * 
        vec3(st.y - st.x * 10.5) *
        (1. - step(0., st.y)) * 
        (1. - step(0., st.x)) * 
        circleFull(length(st - center), 0.64)
        * 0.09;
    
    cplot_1 *= CLR_BLUE;
    cplot_2 *= CLR_BLUE;
    cplot_3 *= CLR_BLUE;
    
    ctarg_1 *= CLR_ORANGE * vec3(0.6);
    ctarg_2 *= CLR_ORANGE;
    ctarg_3 *= CLR_ORANGE;
    
    color += grad * CLR_BLUE;
    
    color += cplot_1;
    color += cplot_2;
    color += cplot_3;
    color += cplot_4;
    color += cplot_5 * (1. - line(st.y, center, 0.25)) * CLR_BLUE * 0.3;
    color += cplot_6 * (1. - line(st.y, center, 0.04)) * CLR_BLUE;
    
    color += ctarg_1;
    color += ctarg_2;
    color += ctarg_3;
    
    color += cfull_1;
  	color += cfull_2;
    
    color += line_1 * 0.3;
    color += line_2 * 0.3;
    color += line_3 * CLR_BLUE;
    
    color += triangle_1;
    color += triangle_2;
    
    color += 0.04;
    
    
    
    gl_FragColor = vec4(color, 1.);
}
```