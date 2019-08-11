package io.swagger.api;

import java.io.File;
import java.io.FileReader;
import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.ws.rs.Consumes;
import javax.ws.rs.HeaderParam;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.Response;
import javax.ws.rs.core.Response.Status;

import com.google.api.client.auth.oauth2.AuthorizationCodeFlow;
import com.google.api.client.auth.oauth2.BearerToken;
import com.google.api.client.auth.oauth2.StoredCredential;
import com.google.api.client.extensions.servlet.auth.oauth2.AbstractAuthorizationCodeServlet;
import com.google.api.client.googleapis.auth.oauth2.GoogleClientSecrets;
import com.google.api.client.googleapis.auth.oauth2.*;
import com.google.api.client.http.BasicAuthentication;
import com.google.api.client.http.GenericUrl;
import com.google.api.client.http.javanet.NetHttpTransport;
import com.google.api.client.json.JsonFactory;
import com.google.api.client.json.jackson2.JacksonFactory;
import com.google.api.client.util.store.FileDataStoreFactory;

import io.swagger.mysql.DataAccess;


@Path("/auth")
@Consumes({ "application/json" })
@Produces({ "application/json" })
public class AuthService  {
	private String REDIRECT_URI = "http://localhost:8080";
	DataAccess dataAccess = DataAccess.getInstance();
	//private final String CLIENT_ID = "673894474162-t0udjug4dro0htn3v3vdl0qrhi2fnt1b.apps.googleusercontent.com";
	//private final String CLIENT_SECRET = "4rBrw6kNdD50gSLRPF6omHjz";
	
	@POST
	public Response authUser(@HeaderParam("X-Requested-With") String requestedHeader, String authResult)
	{	
		//authcode as param
		// (Receive authCode via HTTPS POST)

		System.out.println(authResult);
		if (requestedHeader == null) {
		  // Without the `X-Requested-With` header, this request could be forged. Aborts.
			return Response.status(Status.BAD_REQUEST).entity(new ApiResponseMessage(ApiResponseMessage.ERROR, "The request needs a X-Requested-With Header")).build();
		}

		// Set path to the Web application client_secret_*.json file you downloaded from the
		// Google API Console: https://console.developers.google.com/apis/credentials
		// You can also find your Web application client ID and client secret from the
		// console and specify them directly when you create the GoogleAuthorizationCodeTokenRequest
		// object.
		String CLIENT_SECRET_FILE = "/home/vmuser/Dokumente/client_secret_673894474162-ol5ei74odbet7mbj4i9gpho2us7kl53v.apps.googleusercontent.com.json";

		// Exchange auth code for access token
		GoogleClientSecrets clientSecrets = null;
		try {
			clientSecrets = GoogleClientSecrets.load(
		        JacksonFactory.getDefaultInstance(), new FileReader(CLIENT_SECRET_FILE));
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		GoogleTokenResponse tokenResponse = null;
		try {
			tokenResponse = new GoogleAuthorizationCodeTokenRequest(
			      new NetHttpTransport(),
			      JacksonFactory.getDefaultInstance(),
			      "https://www.googleapis.com/oauth2/v4/token",
			      clientSecrets.getDetails().getClientId(),
			      clientSecrets.getDetails().getClientSecret(),
			      authResult, //authCode
			      REDIRECT_URI)  // Specify the same redirect URI that you use with your web
			                     // app. If you don't have a web version of your app, you can
			                     // specify an empty string.
			      .execute();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		String accessToken = tokenResponse.getAccessToken();
		
		// Get profile info from ID token
		GoogleIdToken idToken = null;
		try {
			idToken = tokenResponse.parseIdToken();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		GoogleIdToken.Payload payload = idToken.getPayload();
		String userId = payload.getSubject();  // Use this value as a key to identify a user.
		//String email = payload.getEmail();
		//boolean emailVerified = Boolean.valueOf(payload.getEmailVerified());
		String name = (String) payload.get("name");
		//System.out.println(name);
		//String pictureUrl = (String) payload.get("picture");
		//String locale = (String) payload.get("locale");
		//String familyName = (String) payload.get("family_name");
		String givenName = (String) payload.get("given_name");
		
		dataAccess.createUser(userId, givenName, "addTokenHere");
		
		return Response.status(Status.OK).entity(dataAccess.getUserById(userId)).build();

	}
}
