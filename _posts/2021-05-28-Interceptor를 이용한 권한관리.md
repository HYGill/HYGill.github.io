## Interceptor를 이용한 권한관리

- 프로젝트 개요 : 포털을 개발하고 있는 중에 사용자의 권한에 맞게 메뉴접근권한을 주고 사용자가 권한이 없는 서비스를 이용하려고 하였을 때 알림을 주어야했다.

  

DB구조가 

User = UserRole = Role = RoleMenu = Menu

User = UserRole = Role = RoleResource = Resource

이렇게 연결된 상태였고 최종적으로 사용자가 서비스를 제공받으려고 할 때 그 api(resource)에 관한 권한이 없는 상태라면

''접근권한이 없습니다'' 라는 메세지와 함께 메인 페이지로 리다이렉트 되게 된다.



```java
@Slf4j
@Component("resourceUrlInterceptorHandle")
public class ResourceUrlInterceptorHandle extends HandlerInterceptorAdapter {

    ...

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String message = "";
        ResourceVo resource = null;

        // 요청한 URL의 Resource 조회
        try {
            resource = resourceService.getByUrl(request.getRequestURI()).getVo(mapper);

            // URI 에 대한 권한 체크
            String roleName = "";

          	...

            // 메소드 검사
            Method nowMethod = null;
            for(Method method : Method.values()){
                if(request.getMethod().equalsIgnoreCase(method.name())){
                    nowMethod = method;
                }
            }

          // roleResource에 접근권한이 있을때
            List<RoleResourceVo> roleResources = roleResourceService.findRoles(
                    RoleResourceFindDto.builder()
                                    .resourceId(resource.getId())
                                    .method(nowMethod)
                                    .roleId(roleId)
                                    .build()
                            .or(RoleResourceFindDto.builder()
                                    .resourceId(resource.getId())
                                    .roleId(roleId)
                                    .method(Method.ALL)
                                    .build()));

            // 해당 리소스에 대한 권한이 없을 때
            if(roleResources.isEmpty()) {
                message = resource.getName() + " 리소스URL을(를) 권한(" + roleName + ")에 추가해주세요";
                throw new ResourceNotAllowException(message);
            } else {
                ...
            }
        } catch(NullPointerException e) {
            // 해당 리소스가 없을때
            message = request.getRequestURI() + "를(을) 리소스URL에서 추가해주세요";
            throw new ResourceNotAllowException(message);
        }

        return super.preHandle(request, response, handler);
    }
}
```



 

이러면 서버에 들어오기 전에 interceptor에서 먼저 처리하므로 간단하게 권한관리가 가능하게 됐다.



인터셉터 구현은 처음이었어서 많이 배우는 계기가 되었다. 끝~!
