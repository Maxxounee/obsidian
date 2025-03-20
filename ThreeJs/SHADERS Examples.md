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