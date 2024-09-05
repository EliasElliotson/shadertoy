# Colored Lighting

This shader implements the idea that lighter colors used for shading should be yellower, and darker colors should be bluer.

## Example

<img src="https://github.com/user-attachments/assets/08cd65f2-c4e2-4285-84c2-5bd261476fdb" alt="alt text" width="49%">
<img src="https://github.com/user-attachments/assets/1e0858cc-bb11-421c-a2e0-c2c58c581a69" alt="alt text" width="49%">

## Shader
```glsl
//gShaderToy.SetTexture(0, {mSrc:'https://corsproxy.io/?https%3A%2F%2Fi.ibb.co%2FKDg8tvX%2FNew-Piskel-1-png-2.png', mType:'texture', mID:1, mSampler:{ filter: 'mipmap', wrap: 'repeat', vflip:'true', srgb:'false', internal:'byte' }});

vec3 lerpColor(vec3 c1, vec3 c2, float amt) {
    return vec3(c1.r + (c2.r - c1.r) * amt, c1.g + (c2.g - c1.g) * amt, c1.b + (c2.b - c1.b) * amt);
}

vec3 hsl2rgb(in vec3 c) {
    vec3 rgb = clamp( abs(mod(c.x*6.0+vec3(0.0,4.0,2.0),6.0)-3.0)-1.0, 0.0, 1.0 );

    return c.z + c.y * (rgb-0.5)*(1.0-abs(2.0*c.z-1.0));
}

vec3 rgb2hsl(in vec3 c) {
  float h = 0.0;
	float s = 0.0;
	float l = 0.0;
	float r = c.r;
	float g = c.g;
	float b = c.b;
	float cMin = min( r, min( g, b ) );
	float cMax = max( r, max( g, b ) );

	l = ( cMax + cMin ) / 2.0;
	if ( cMax > cMin ) {
		float cDelta = cMax - cMin;
        
        //s = l < .05 ? cDelta / ( cMax + cMin ) : cDelta / ( 2.0 - ( cMax + cMin ) ); Original
		s = l < .0 ? cDelta / ( cMax + cMin ) : cDelta / ( 2.0 - ( cMax + cMin ) );
        
		if ( r == cMax ) {
			h = ( g - b ) / cDelta;
		} else if ( g == cMax ) {
			h = 2.0 + ( b - r ) / cDelta;
		} else {
			h = 4.0 + ( r - g ) / cDelta;
		}

		if ( h < 0.0) {
			h += 6.0;
		}
		h = h / 6.0;
	}
	return vec3( h, s, l );
}

vec3 colorizeShading(vec3 col) {
    vec3 colHsl = rgb2hsl(col);
    vec3 hueCol = hsl2rgb(vec3(colHsl.x, 1.0, 0.5));
    
    vec3 white = vec3(1.0, 1.0, 1.0);
    vec3 yellow = vec3(1.0, 1.0, 0.0);
    vec3 blue = vec3(0.0, 0.0, 1.0);
    vec3 black = vec3(0.0, 0.0, 0.0);
    
    vec3 lightColor = lerpColor(yellow, white, (colHsl.z - 0.5) * 2.0);
    vec3 darkColor = lerpColor(blue, black, -(colHsl.z - 0.5) * 2.0);
    
    vec3 shadeColor = lightColor;
    
    if (colHsl.z < 0.5) {
        shadeColor = darkColor;
    }
    
    vec3 fullColor = lerpColor(hueCol, shadeColor, abs((colHsl.z - 0.5) * 2.0));
    
    vec3 finalColor = rgb2hsl(fullColor);
    
    finalColor.y = colHsl.y;
    
    return hsl2rgb(finalColor);
}

void mainImage( out vec4 fragColor, in vec2 fragCoord ) {
    // Normalized pixel coordinates (from 0 to 1)
    vec2 uv = fragCoord/iResolution.xy;

    // Time varying pixel color
    vec3 col = colorizeShading(texture(iChannel0, uv).rgb);

    // Output to screen
    fragColor = vec4(col.r, col.g, col.b, 1.0);
}
```
