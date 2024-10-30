# IMPLEMENTAR NOTACION PERSONALIZADA  @JsonArg en Spring Boot  ( Acepta múltiples variables a un Controlador  usando un solo @RequestBody )
Manuales de integracion y mejoras de servicios C2U

## PASO 1 
  Crear y agregar la clase interface JsonArg la paquete  `ruta-proyecto/configuration.parameters`

```java	
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

import java.lang.annotation.Target;

@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface JsonArg {
    String value() default "";
}
```
## PASO 2
  Crear y agregar la clase java JsonPathArgumentResolver la paquete  `ruta-proyecto/configuration.parameters`

```java	

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.io.IOUtils;
import org.springframework.core.MethodParameter;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

public class JsonPathArgumentResolver implements HandlerMethodArgumentResolver{

    private static final String JSONBODYATTRIBUTE = "JSON_REQUEST_BODY";

    private ObjectMapper om = new ObjectMapper();

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(JsonArg.class);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        String jsonBody = getRequestBody(webRequest);

        JsonNode rootNode = om.readTree(jsonBody);
        JsonNode node = rootNode.path(parameter.getParameterName());

        return om.readValue(node.toString(), parameter.getParameterType());
    }


    private String getRequestBody(NativeWebRequest webRequest){
        HttpServletRequest servletRequest = webRequest.getNativeRequest(HttpServletRequest.class);

        String jsonBody = (String) webRequest.getAttribute(JSONBODYATTRIBUTE, NativeWebRequest.SCOPE_REQUEST);
        if (jsonBody==null){
            try {
                jsonBody = IOUtils.toString(servletRequest.getInputStream());
                webRequest.setAttribute(JSONBODYATTRIBUTE, jsonBody, NativeWebRequest.SCOPE_REQUEST);
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
        return jsonBody;

    }

}
```

## PASO 3 
  Crear y agregar la clase java WebConfig al paquete configuration u otra más conveniente en el proyecto en el que se este trabajando   `ruta-proyecto/configuration`

```java	
import org.springframework.context.annotation.Configuration;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import pe.close2u.transportapi.configuration.parameters.JsonPathArgumentResolver;

import java.util.List;

@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(new JsonPathArgumentResolver());
    }
}
```
## Forma de utilizar la notacion @JsonArg 
Luego de habilitar la notación , se puede utilizar en la definición de nuestro paquete alma en la configuración del parametro de entrada de nuestro endpoint. 

![Referencia Imagen1](https://raw.githubusercontent.com/BOC-DYAN/Integraciones/b1cfd842cf1e973b4c5b99ce921294685a041bd4/images-manual/Paso1.png)

IMPORTANTE . En la implmentación del metodo Controller se debe definir la variable igual al atributo que se intenta mapear desde el contenido del JSON. 

![Referencia Imagen2](https://raw.githubusercontent.com/BOC-DYAN/Integraciones/refs/heads/main/images-manual/Paso2.png)

El resultado es que la notacion JsonArg nos permite mappear multiples secciones de una misma entrada y manipular el tipo de variable que corresponda String , Integer , 

![Referencia Imagen2](https://raw.githubusercontent.com/BOC-DYAN/Integraciones/refs/heads/main/images-manual/Paso3.png)


Referencias : 

https://stackoverflow.com/questions/12893566/passing-multiple-variables-in-requestbody-to-a-spring-mvc-controller-using-ajax
https://www.aluracursos.com/blog/crear-anotaciones-en-java
https://www.geekabyte.io/2014/08/how-to-inject-objects-into-spring-mvc.html


