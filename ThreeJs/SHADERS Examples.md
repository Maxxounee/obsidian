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

