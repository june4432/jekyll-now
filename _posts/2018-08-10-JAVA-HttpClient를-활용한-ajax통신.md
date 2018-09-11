프로젝트를 진행하면서 클라이언트단이 아닌 서버단에서 SMS서버와 AJAX통신을 해야해서 여러 코드를 찾던 중 자바버전이 올라가면서
참조할 만한 코드들이 존재하지 않게 되어 HttpClient를 활용한 ajax통신이 가능한 메소드를 만들게 되었습니다.

코드는 몇가지 단계로 구분이 됩니다.
1. 화면에서 받아온 파라미터를 통해 jsonObject를 생성합니다.
2. 커넥션설정을 합니다.
3. 전송 및 리턴값을 받아 정규식으로 치환합니다. 
   (이 프로젝트에서는 결과값이 16진수로 리턴값이 와서 리턴메세지가 짤리는 현상이 발생하여 정규식으로 모두 치환을 했습니다.)
4. 업무테이블에 결과 저장

소스는 아래와 같으니 참조하시면 됩니다.

public void postSms(String C_CD, String TRG_EMP_ID, String FROM_TEL_NO, String TO_TEL_NO, String MESSAGE) throws SQLException, hunelBizException, IOException, JSONException
{
		//테스트용 URL 및 파라미터
		String URL = "호출할url";
		String API_KEY = "API_KEY";
    
    String msg = "{\"mms_subject\":\"자바TEST입니다!!\",\"tran_Phone\":\"수신번호\",\"tran_MSG\":\"test\",\"tran_CallBack\":\"송신번호\"}";
 
		//전송할 JSON 객체를 만들어준다.
		String jsonStr = "";
		JSONObject jo = new JSONObject();
		JSONArray ja = new JSONArray();
		
		jo.put("tran_Phone", TO_TEL_NO); //발신자
		jo.put("tran_CallBack", FROM_TEL_NO); //수신자
		jo.put("tran_MSG", MESSAGE); //내용
		
		//80바이트 초과시 MMS로 전환
		if(MESSAGE.getBytes().length > 80){
			jo.put("tran_Type", "6"); //전송타입 6 = MMS
			jo.put("mms_subject", ""); //제목
			jo.put("file_Type1", ""); //파일타입
			jo.put("file_Name1", ""); //주소
		}else
		{
			jo.put("tran_Type", "4"); //전송타입 4 = SMS
			jo.put("mms_subject", ""); //제목
			jo.put("file_Type1", ""); //파일타입
			jo.put("file_Name1", ""); //주소
		}
		
		ja.add(jo);
		
		//JSON오브젝트를 스트링으로 변환.
		
		jsonStr = ja.toString();
		
		log.debug(jsonStr);
		
		//POST 전송 준비....
		HttpResponse httpresponse = null;
		String returnJaStr = "";
		
		try {
		
		  
			CloseableHttpClient httpclient = HttpClients.createDefault();
			Builder builder = RequestConfig.custom();
			builder.setConnectTimeout(4000);
			builder.setSocketTimeout(4000);
			builder.setStaleConnectionCheckEnabled(false);
			RequestConfig config = builder.build();
			
			try {
		
				//URL CONNECTION START..
				HttpPost httpPost = new HttpPost(URL);
				//전송할 헤더설정
				httpPost.addHeader("APIKey", API_KEY);
				httpPost.addHeader("Content-Type", "application/json");
				httpPost.addHeader("Accept","application/json");
				httpPost.addHeader("Accept-Charset","UTF-8");
				
				//json 형태의 스트링을 StringEntity로 변환
				StringEntity jsonInput = new StringEntity(jsonStr,"UTF-8");
				httpPost.setEntity(jsonInput);
				
				//PRINT LOG...
				/*log.debug("jsonStr : " + jsonStr);
				log.debug("url : " + URL);
				log.debug("Executing request " + httpPost.getRequestLine());*/
				
				//POST로 전송...
				httpresponse = httpclient.execute(httpPost);
				
				//log.debug("GET STATUS LINE" + httpresponse.getStatusLine().toString());
				
				//리턴값 출력
				int i;
				InputStream is = httpresponse.getEntity().getContent();
				StringBuffer buffer = new StringBuffer();
				byte[] b = new byte[4096];
				while( (i = is.read(b)) != -1){
					buffer.append(new String(b, 0, i));
				}
				
				//log.debug("USING INPUTSTREAM ==== " + httpresponse.getEntity().getContent().toString());
				//log.debug("USING INPUTSTREAM ==== " + buffer.toString());
				
				//정규식을 통해 공백문자, 개행문자, [, ]  제거 후 JSONArray 형태의 스트링을 JSONObject String으로 변경
				returnJaStr = buffer.toString();
				returnJaStr = returnJaStr.replaceAll("[^0-9a-zA-Z:\\{\\}\"]", "");
				//log.debug(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>     " + returnJaStr);
				
				//JSONOBJECT형태의 스트링을 JSONOBJECT로 변경
				JSONObject jo2 = new JSONObject(returnJaStr);
				
				//JSONARRAY에서 JSONOBJECT 추출
				String returnNumber = jo2.getString("code");
				log.debug("returnNumber ===== " + returnNumber);
				
				String successYn = "";
				
				if(returnNumber.equals("success")) {
					successYn = "Y";
				}else {
					successYn = "N";
				}
				
        //업무테이블에 전송결과 저장 메소드
		
				} finally {
					if(httpclient != null)httpclient.close();
			}
			
		} catch(Exception e) {
			log.debug(e);
		}
    }
