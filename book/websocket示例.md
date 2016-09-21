��ǰǰ��˽�����Ҫ��֤�������ã�������׼ʵʱ�Ĵ���ҵ�񣬲��õ�������ͨ��JS��һ���̣߳���ͣ��ȥ��ѭ�����ˣ��Ѵﵽ׼ʵʱ����ҵ���Ŀ��.`websocket`�ĳ��֣��ܺõĽ�����������������ӳ٣�ǰ�θ��ؽϴ���������������죬��̨ѹ��������⣬ͨ��ȫ˫��ͨ�Ż��ƣ�����ǰ��˵Ľ����������ҵ���������������͸�ǰ̨������Ҫǰ̨��ͣ���˷���Դȥ����ѭ�����ܴﵽ�ϸߵ�ʵʱ�Դ���(���ų������ӳ�).

`spring`��4.0��ʼ��`websocket`�кܺõ�֧�֣�����ͨ��`spring`������ʾ��`websocket`��ʵ�ֹ���,��Ȼ`spring`���˴����Ĺ���.
#`spring`����`websocket`
##��Ҫ����
```java
        <dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-websocket</artifactId>
			<version>${springversion}</version>
		</dependency>
```
##����websocket������
websocket��������Ҫʵ��`WebSocketHandler`��һЩ������أ���Ϣ���������,�������棺
```java
public class LogWebSocketHandler implements WebSocketHandler {

	private static final ArrayList<WebSocketSession> users = new ArrayList<WebSocketSession>();

	@Override
	public void afterConnectionEstablished(WebSocketSession session) throws Exception {
		System.out.println("ConnectionEstablished");
		users.add(session);
        Map<String,Object> map=session.getAttributes();
        for(Entry<String,Object> entry:map.entrySet()){
        	System.out.println(entry.getKey()+"="+entry.getValue());
        }
		session.sendMessage(new TextMessage("connect"));
		session.sendMessage(new TextMessage("new_msg"));

	}

	@Override
	public void handleMessage(WebSocketSession session, WebSocketMessage<?> message) throws Exception {
		session.sendMessage(new TextMessage(new Date() + ""));
	}

	@Override
	public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
		if (session.isOpen()) {
			session.close();
		}
		users.remove(session);
		System.out.println("handleTransportError" + exception.getMessage());
	}

	@Override
	public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception {
		users.remove(session);
		System.out.println("afterConnectionClosed" + closeStatus.getReason());

	}

	@Override
	public boolean supportsPartialMessages() {
		return false;
	}

	/**
	 * �����������û�������Ϣ
	 * 
	 * @param message
	 */
	public void sendMessageToUsers(TextMessage message) {
		for (WebSocketSession user : users) {
			System.out.println(user.getAttributes().get("USER_ID"));
			try {
				if (user.isOpen()) {
					user.sendMessage(message);
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

	/**
     * ��ĳ���û�������Ϣ
     *
     * @param userName
     * @param message
     */
    public void sendMessageToUser(String anoyUserId, TextMessage message) {
        for (WebSocketSession user : users) {
            if (user.getAttributes().get("USER_ID").equals(anoyUserId)) {
                try {
                    if (user.isOpen()) {
                        user.sendMessage(message);
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
                break;
            }
        }
    }
}

```
##����websocket����������
websocket������������Ҫʵ��`HandshakeInterceptor`��ʵ�ֽ�������ʱ��һЩ���ԣ��������صȴ���.
```java

public class HandshakeInterceptorImpl implements HandshakeInterceptor {

	@Override
	public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler handler,
			Exception exception) {

	}

	@Override
	public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler handler,
			Map<String, Object> map) throws Exception {
		if (request instanceof ServletServerHttpRequest) {
			ServletServerHttpRequest servletRequest = (ServletServerHttpRequest) request;
			/*
			 * getSession����:
			 * Ϊtrue�ȼ��ڲ���ֵ����ʾ��û��sessionʱ����һ���µ�.
			 * Ϊfalse��ʾ��û��sessionʱ������
			 */
			HttpSession session = servletRequest.getServletRequest().getSession(true);
			if (session != null) {
				String anoyUserId = session.getId();
				map.put("USER_ID", anoyUserId);
			}
			return true;
		}
		return false;
	}
}
```
##ʵ��������websocket����
webcsocket������Ҫʵ��`WebSocketConfigurer`��ע�ᴦ����ƣ�����·��ƥ�䣬������������Ϣ�����.
```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {
	public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
		registry.addHandler(systemWebSocketHandler(), "/webSocketServer")
				.addInterceptors(new HandshakeInterceptorImpl());
		registry.addHandler(systemWebSocketHandler(), "/webSocketServer/sockjs")
				.addInterceptors(new HandshakeInterceptorImpl()).setAllowedOrigins("*").withSockJS();
	}

	@Bean
	public WebSocketHandler systemWebSocketHandler() {
		return new LogWebSocketHandler();
	}
}
```
����ע���˲�ͬ����·����Ӧ�Ĳ�ͬ������,�Լ���Ӧ�����ش�����.
��Ȼ����ͨ��`@Configuration`��ʶ������Ϊbean����,`@EnableWebSocket`��ʶ��һ��`WebSocket`����ʵ�֣�������Ҫ��spring������ɨ�赽����ļ�,��
```java
<context:component-scan base-package="com.wyp.module.socketlink" />
```
��ɨ�����ڰ�.


```java
registry.addHandler(systemWebSocketHandler(), "/webSocketServer")
                .addInterceptors(new HandshakeInterceptorImpl());
```
������ô���`websocket`����Ҫ�����֧��`websocket`.`systemWebSocketHandler`������,`HandshakeInterceptorImpl`���������ش���.
```java
registry.addHandler(systemWebSocketHandler(), "/webSocketServer/sockjs")
                .addInterceptors(new HandshakeInterceptorImpl()).setAllowedOrigins("*").withSockJS();
```
������Դ����������֧��websocket�������ͨ��`socketjs`����`AllowedOrigins`��ʾ�������е���Դ.

##ǰ��`WebSocket`����
```javascript
//ֱ��WebSocket
              //ws = new WebSocket('ws://127.0.0.1:8080/WebsocketTest/webSocketServer/');
              //SockJS����
              ws = new SockJS("http://127.0.0.1:8080/WebsocketTest/webSocketServer/sockjs");
                
            ws.onopen = function () {
                setConnected(true);
                log('Info: connection opened.');
            };
            
            ws.onmessage = function (event) {
                log('Received: ' + event.data);
            };
            
            ws.onclose = function (event) {
                setConnected(false);
                log('Info: connection closed.');
                log(event);
            };
```
��Ȼ��Ҫ����js�ļ�
```javascript
<script src="http://cdn.sockjs.org/sockjs-0.3.min.js"></script>
```
##���������ʾ��
ͨ�������۲�����ͷ��Ϣ��ͨ��`ws`����ʶһ��websocket����.����ͼ��

![](../image/ws.png)

##һЩ����
�ڴ����У����ǿ����ں�̨����ҵ������������ǰ̨��һЩ����,������������ĳ����ͨ��һ��ʱ����ǰ̨����һ������
```java
@Bean
	public LogWebSocketHandler logWebSocketHandler(){
		return new LogWebSocketHandler();
	}
	
@RequestMapping(value = "/dosomething", method = RequestMethod.GET)
	public String userLogin(HttpServletRequest request) {
		logWebSocketHandler().sendMessageToUsers(new TextMessage("����" + request.getSession().getId()));
		new Thread() {
			public void run() {
				int i=0;
				while(i<20){
					try {
						logWebSocketHandler().sendMessageToUsers(new TextMessage("���ǵ� "+i+" ����ǰ̨������Ϣ"));;
						i++;
						Thread.sleep(1000);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			};
		}.start();
		return "index.jsp";
	}

```