from smb.SMBConnection import SMBConnection

def check_file_exists(smb_server, smb_share, directory, file_name, username, password, domain='', use_ntlm_v2=True):
    # Kết nối tới SMB server
    conn = SMBConnection(username, password, 'my_client', smb_server, domain=domain, use_ntlm_v2=use_ntlm_v2)
    conn.connect(smb_server, 139)

    try:
        # List tất cả các file trong thư mục
        files = conn.listPath(smb_share, directory)

        # Kiểm tra sự tồn tại của file
        for file in files:
            if file.filename == file_name:
                print(f"File '{file_name}' exists in the path '{directory}'")
                return True

        print(f"File '{file_name}' does not exist in the path '{directory}'")
        return False
    except Exception as e:
        print(f"An error occurred: {e}")
        return False
    finally:
        # Đóng kết nối SMB
        conn.close()

# Sử dụng hàm kiểm tra
smb_server = 'your_smb_server'   # Thay bằng địa chỉ server của bạn
smb_share = 'your_share_name'    # Thay bằng tên share của bạn (vd: test)
directory = 'csv/2024'           # Thư mục chứa file abc.csv
file_name = 'abc.csv'            # Tên file bạn muốn kiểm tra
username = 'your_username'       # Tên đăng nhập SMB
password = 'your_password'       # Mật khẩu SMB

file_exists = check_file_exists(smb_server, smb_share, directory, file_name, username, password)
