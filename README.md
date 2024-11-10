
Mô tả : Nhìn qua source code tôi gửi ở trên có thể thấy ta sẽ lần lượt vào các điểm cuối để thực hiện thao tác ở đây ta chú ý /register có phần kiểm tra nếu có chữ admin thì sẽ block. Vì thế ta sẽ tìm cách bypass 

            @app.route('/register', methods=['POST'])
    def register():
        json_data = request.data
        if "admin" in json_data:
            return jsonify(message="Blocked!")
        data = ujson.loads(json_data)
        username = data.get('username')
        password = data.get('password')
        role = data.get('role')
    
    if role !="admin" and role != "user":
        return jsonify(message="Never heard about that role!")
    
    if username == "" or password == "" or role == "":
        return jsonify(messaage="Lack of input")
    
    if register_db(connection, username, password, role):
        return jsonify(message="User registered successfully."), 201
    else:
        return jsonify(message="Registration failed!"), 400
ở đây ta thấy data sẽ dùng hàm ujson.load() đây là một hàm dễ bị tấn công để hiểu rõ hơn ta có thể tìm hiểu ở bài viết https://github.com/ultrajson/ultrajson/issues/11. 
![image](https://github.com/user-attachments/assets/3317fe5b-9b25-4e84-a4c1-e6a243ace5f1)
ở đây ta có thể bypass chữ admin bằng cách như thế và sẽ đăng ký thành công ko bị chặn với quyền admin
Từ đây ta chứ ý endpoint /render. 

        @app.route('/render', methods=['POST'])
        def dynamic_template():
            token = request.cookies.get('jwt_token')
            if token:
                try:
                    decoded = jwt.decode(token, app.config['SECRET_KEY'], algorithms=['HS256'])
                    role = decoded.get('role')
        
                    if role != "admin":
                        return jsonify(message="Admin only"), 403
        
                    data = request.get_json()
                    template = data.get("template")
                    rendered_template = render_template_string(template)
                    
                    return jsonify(message="Done")
        
                except jwt.ExpiredSignatureError:
                    return jsonify(message="Token has expired."), 401
                except jwt.InvalidTokenError:
                    return jsonify(message="Invalid JWT."), 401
                except Exception as e:
                    return jsonify(message=str(e)), 500
            else:
                return jsonify(message="Where is your token?"), 401
Ở đây ta sẽ hiểu là code này sẽ sử dụng data của data của request chúng ta gửi lên là lấy header json template trong đó để thực hiện hàm rrender. Như vậy ta thấy có thể khai thách lỗ hổng SSTI ở đây . Payload của tôi sẽ là như sau:
![image](https://github.com/user-attachments/assets/aef8e966-f850-40a1-ad18-3725e8e1e935)
Ở đây có nghĩa là tôi sẽ chèn một template có thể thực hiện một số lệnh thực thi cụ thể là nhìn vào file docker ta có thể thấy là web này đang hoạt động trên thư mục /app nên ta có thể tạo một thư mục khác có tên là static đồng thời thực hiện thêm một hàm là cat * > /app/static/test , sau đó thì ta chỉ cần vào đường dẫn đó là sẽ thấy flag hiện ra thôi
![image](https://github.com/user-attachments/assets/c90329bc-2f85-4410-a504-5967326a7591)



        



