## Introdução aos Servlets Java

# 1. Visão Geral
Neste artigo, daremos uma olhada em um aspecto central do desenvolvimento web em Java - Servlets.

# 2. O servlet e o recipiente
Simplificando, um Servlet é uma classe que lida com solicitações, processa-as e responde com uma resposta.

Por exemplo, podemos usar um Servlet para coletar dados de um usuário por meio de um formulário HTML, consultar registros de um banco de dados e criar páginas da web dinamicamente.

Os servlets estão sob o controle de outro aplicativo Java chamado Container de Servlet. Quando um aplicativo em execução em um servidor da web recebe uma solicitação, o servidor entrega a solicitação ao contêiner de servlet - que, por sua vez, a transmite ao servlet de destino.

# 3. Dependências Maven
Para adicionar suporte a Servlet em nosso aplicativo da web, a dependência javax.servlet-api é necessária:

```
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
</dependency>
```

Claro, também teremos que configurar um contêiner de Servlet para implantar nosso aplicativo; este é um bom lugar para começar a implantar um WAR no Tomcat.

# 4. Ciclo de vida do servlet
Vamos examinar o conjunto de métodos que definem o ciclo de vida de um Servlet.

### 4.1. init()
O método init foi projetado para ser chamado apenas uma vez. Se uma instância do servlet não existir, o contêiner da web:

- Carrega a classe servlet;
- Cria uma instância da classe servlet;
- Inicializa chamando o método init.

O método init deve ser concluído com sucesso antes que o servlet possa receber qualquer solicitação. O contêiner de servlet não pode colocar o servlet em serviço se o método init lançar uma ServletException ou não retornar dentro de um período de tempo definido pelo servidor da web.

```
public void init() throws ServletException {
    // Initialization code like set up database etc....
}
```

### 4.2. service()
Este método só é chamado depois que o método init () do servlet foi concluído com sucesso.

O Container chama o método service () para lidar com solicitações vindas do cliente, interpreta o tipo de solicitação HTTP (GET, POST, PUT, DELETE, etc.) e chama os métodos doGet, doPost, doPut, doDelete etc. conforme apropriado.

```
public void service(ServletRequest request, ServletResponse response) 
  throws ServletException, IOException {
    // ...
}
```

### 4.3. destroy()
Chamado pelo contêiner de servlet para tirar o servlet de serviço.

Este método só é chamado depois que todos os threads dentro do método de serviço do servlet foram encerrados ou após um período de tempo limite ter passado. Depois que o contêiner chamar esse método, ele não chamará o método de serviço novamente no Servlet.

```
public void destroy() {
    // 
}
```

# 5. Exemplo de servlet
Vamos agora configurar um exemplo completo de manipulação de informações usando um formulário.

Para começar, vamos definir um servlet com um mapping /calculateServlet which que irá capturar as informações POSTed pelo formulário e retornar o resultado usando um RequestDispatcher:

```
@WebServlet(name = "FormServlet", urlPatterns = "/calculateServlet")
public class FormServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request, 
      HttpServletResponse response)
      throws ServletException, IOException {

        String height = request.getParameter("height");
        String weight = request.getParameter("weight");

        try {
            double bmi = calculateBMI(
              Double.parseDouble(weight), 
              Double.parseDouble(height));
            
            request.setAttribute("bmi", bmi);
            response.setHeader("Test", "Success");
            response.setHeader("BMI", String.valueOf(bmi));

            RequestDispatcher dispatcher 
              = request.getRequestDispatcher("index.jsp");
            dispatcher.forward(request, response);
        } catch (Exception e) {
            response.sendRedirect("index.jsp");
        }
    }

    private Double calculateBMI(Double weight, Double height) {
        return weight / (height * height);
    }
}
```

Conforme mostrado acima, as classes anotadas com @WebServlet devem estender a classe 
javax.servlet.http.HttpServlet. É importante observar que a anotação @WebServlet está disponível apenas a partir do Java EE 6 em diante.

A anotação @WebServlet é processada pelo contêiner no momento da implementação e o servlet correspondente é disponibilizado nos padrões de URL especificados. É importante notar que, usando a anotação para definir padrões de URL, podemos evitar o uso do descritor de implantação XML denominado web.xml para nosso mapeamento de Servlet.

Se desejarmos mapear o Servlet sem anotação, podemos usar o web.xml tradicional em seu lugar:

```
<web-app ...>

    <servlet>
       <servlet-name>FormServlet</servlet-name>
       <servlet-class>com.root.FormServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>FormServlet</servlet-name>
        <url-pattern>/calculateServlet</url-pattern>
    </servlet-mapping>

</web-app>
```

A seguir, vamos criar um formulário HTML básico:

```
<form name="bmiForm" action="calculateServlet" method="POST">
    <table>
        <tr>
            <td>Your Weight (kg) :</td>
            <td><input type="text" name="weight"/></td>
        </tr>
        <tr>
            <td>Your Height (m) :</td>
            <td><input type="text" name="height"/></td>
        </tr>
        <th><input type="submit" value="Submit" name="find"/></th>
        <th><input type="reset" value="Reset" name="reset" /></th>
    </table>
    <h2>${bmi}</h2>
</form>
```

Finalmente - para ter certeza de que tudo está funcionando conforme o esperado, também vamos escrever um teste rápido:

```
public class FormServletLiveTest {

    @Test
    public void whenPostRequestUsingHttpClient_thenCorrect() 
      throws Exception {

        HttpClient client = new DefaultHttpClient();
        HttpPost method = new HttpPost(
          "http://localhost:8080/calculateServlet");

        List<BasicNameValuePair> nvps = new ArrayList<>();
        nvps.add(new BasicNameValuePair("height", String.valueOf(2)));
        nvps.add(new BasicNameValuePair("weight", String.valueOf(80)));

        method.setEntity(new UrlEncodedFormEntity(nvps));
        HttpResponse httpResponse = client.execute(method);

        assertEquals("Success", httpResponse
          .getHeaders("Test")[0].getValue());
        assertEquals("20.0", httpResponse
          .getHeaders("BMI")[0].getValue());
    }
}
```

# 6. Servlet, HttpServlet e JSP
É importante entender que a tecnologia Servlet não se limita ao protocolo HTTP.

Na prática, quase sempre é, mas Servlet é uma interface genérica e o HttpServlet é uma extensão dessa interface - adicionando suporte específico a HTTP - como doGet e doPost, etc.

Finalmente, a tecnologia Servlet também é o principal driver de uma série de outras tecnologias da web, como JSP - JavaServer Pages, Spring MVC, etc.

# 7. Conclusão
Neste artigo rápido, apresentamos os fundamentos dos Servlets em um aplicativo da web Java.