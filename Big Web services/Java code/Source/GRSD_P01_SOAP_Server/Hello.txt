package grsd_p01_soap_server;

import javax.jws.WebService;
import javax.jws.WebMethod;
import javax.jws.WebParam;

// Correr na VM1.
@WebService(serviceName = "Hello")
public class Hello {

    /**
     * @param pac
     */
    @WebMethod(operationName = "hello")
    public void hello(@WebParam(name = "Packet") String pac) {
    }
}
