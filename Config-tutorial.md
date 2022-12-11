<h1>Cài đặt Kubernetes trên instance AWS EC2 Ubuntu 18.04</h1>

<h2>Điều kiện tiên quyết</h2>
<p>Để có thể hoàn thành được quá trình cài đặt, chúng ta cần có một số điều kiện sau: 
<ul>
  <li>Có một tài khoản AWS</li>
  <li>Hiểu biết căn bản về Docker</li>
</ul>
Nếu mọi điều kiện đã sẵn sàng thì chúng ta có thể bắt đầu
</p>

<h2>Cách đọc hướng dẫn cài đặt hiệu quả</h2>
<p>Hướng dẫn sẽ cũng cấp từng bước và các lệnh chi tiết cần phải được thực hiện tuần tự như được liệt kê. Khuyến khích thực hiện đầy đủ hết tất cả các bước, không nên bỏ qua bước nào, ngoại trừ <i>bước giải pháp</i> khi bạn không gặp bất kỳ lỗi nào trong các bước căn bản của quá trình cấu hình.Nếu bạn cảm thấy thuận tiện hơn khi theo dõi qua video thì nhóm có đề xuất một phiên bản video của bài hướng dẫn <a href="https://youtu.be/SQJxYhfzznU">tại đây</a>.</p>

<h3>Cấu trúc của hướng dẫn</h3>
<p>Hướng dẫn này bao gồm các bước sau mà bạn có thể thực hiện tuần tự
<ul>
  <li><b>Bước 1.</b> Thiết lập AWS EC2 Instance Ubuntu 18.04.</li>
  <li><b>Bước 2.</b> Kết nối với EC2 Instance từ terminal trên máy tính cục bộ.
    <ul>
      <li><b>2.1.</b> Tạo khóa riêng của bạn (<i>.pem</i>) thực thi được.</li>
      <li><b>2.2.</b> Kết nối với tất cả các node từ các phiên bản.</li>
    </ul>
  </li>
  <li><b>Bước 3.</b> Cài đặt các dependencies và cấu hình Kubernetes.
    <ul>
      <li><b>3.1.</b> Cập nhật danh sách gói.</li>
      <li><b>3.2.</b> Cài đặt Docker.</li>
      <li><b>3.3.</b> Kiểm tra phiên bản Docker.</li>
      <li><b>3.4.</b> Bắt đầu và kích hoạt Docker.</li>
    </ul>  
  </li>
  <li><b>Bước 4 4.</b> Caià đặt Kubernets trên Ubuntu 18.04.
    <ul>
      <li><b>4.1.</b> Thêm khóa ký (<i>GPG</i>).</li>
      <li><b>4.2.</b> Thêm kho phần mềm Kubernetes.</li>
    </ul>
  </li>
  <li><b>Bước 5 5.</b> Chuẩn bị công cụ cài đặt Kubernetes.
    <ul>
      <li><b>5.1.</b> Cài đặt <i>kubeadm</i>, <i>kubelet</i>, và <i>kubectl</i>.</li>
      <li><b>5.2.</b> Kiểm tra liệu <i>kubeadm</i>, <i>kubelet</i>, và <i>kubectl</i> có đang bị treo.</li>
    </ul>  
  </li>
  <li><b>Bước 6.</b> Triển khai Kubernetes.
     <ul>
      <li><b>6.1.</b> Vô hiệu hóa bộ nhớ trao đổi từ các node.</li>
      <li><b>6.2.</b> Đặt tên máy chủ cho mỗi node.</li>
    </ul> 
  </li>
  <li><b>Bước 7.</b> Giải pháp để tránh sự không phù hợp giữa các trình điều khiển Docker (<i>bước tùy chọn</i>).
     <ul>
       <li><b>7.1.</b> Tạo một file <i>daemon.json</i>.</li>
       <li><b>7.2.</b> Cài lại Docker và dịch vụ <i>kubeadm</i>.</li>
    </ul>   
  </li>
  <li><b>Bước 8.</b> Khởi tạo mạng Kubernetes Pod.
    <ul>
      <li><b>8.1</b> Thiết lập Mạng Pod cho node Master.</li>
      <li><b>8.2</b> Tạo một thư mục cho node Master đã khởi tạo.</li>
      <li><b>8.3</b> Thiết lập một mạng ảo để liên lạc qua các node (<i>flannel</i>).</li>
      <li><b>8.4</b> Kiểm tra trạng thái pod</li>
      <li><b>8.5.</b> Tham gia các node Worker với một node Master</li>
    </ul>  
  </li>
  <li><b>Bước 9 9.</b> Kiểm tra giai đoạn cuối.
    <ul>
      <li><b>9.1</b> Xác nhận rằng node Master giao tiếp với tất cả các node Worker.</li>
    </u>
  </li>
</ul>
</p>

<h2>Bước 1. Thiết lập AWS EC2 Instance Ubuntu 18.04.</h2>
<p>Trong bước này, chúng ta sẽ làm việc chặt chẽ trên các phiên bản AWS EC2 đã khởi tạo. Hãy nhớ rằng để hoàn thành phần này thành công, bạn phải mở tất cả 3 cửa sổ terminal trong tất cả các bước. Ở đây 1 cửa sổ terminal là cho <b>Master</b>, trong khi 2 cái khác cho <b>Worker</b>. Vì vậy, tổng cộng chúng ta có 3 node (1 + 2).<br>Lưu ý rằng sách hướng dẫn này được chuẩn bị cho Mac, vì vậy có thể có một số khác biệt nhỏ khi sử dụng các HĐH khác nhau, chẳng hạn như Windows.</p>
  
<h3>1.1. Chuẩn bị khóa riêng SSH của bạn để kết nối an toàn</h3>
<p>Trước hết, hãy đặt khóa SSH của bạn vào một thư mục đặc biệt trong máy của bạn. Giữ cho tên tệp khóa SSH dễ nhập và dễ nhận biết, chẳng hạn như <i>kubernetes-dev.pem</i> (chúng tôi sẽ sử dụng tên tệp này hơn nữa trong sách hướng dẫn này).</p>
<p>Trước tất cả các bước chuyển tiếp, bạn phải cho phép tệp <i>.pem</i> có thể thực thi được trong hệ thống của mình. Để làm điều này, gõ và nhập lệnh sau:</p>

```
chmod 400 kubernetes-dev.pem 
```

Kết quả là bạn sẽ thấy quyền truy cập được cấp cho tệp <i>.pem</i> của mình - hãy nhập lệnh <code>ls -la</code> để xem kết quả:

```commandline
-r--------@    1 usename  staff   1700 Nov 16 07:14 kubernetes-dev.pem
```

<p><b>Quan trọng:</b> Thay thế mẫu <i>Public IP</i> bằng IP của bạn trong command. 
Vui lòng tạo kết nối tới tất cả các node <b>3</b> bằng cách thay đổi <i>Public IP</i> tương ứng.</p>

<h2>2. Kết nối với phiên bản Ubuntu của bạn từ terminal</h3>
<p>Khi bạn đã tạo khóa riêng (<i>.pem</i>) có thể thực thi được, bạn đã sẵn sàng sử dụng nó để tạo kết nối với phiên bản Ubuntu từ terminal của mình. Bạn có thể làm điều đó bằng lệnh sau:</p>

```
ssh -i kubernetes-dev.pem ubuntu@18.117.218.51
```

Cài đặt này đảm bảo kết nối an toàn với các phiên bản để thực hiện các bước tiếp theo. 
</p>

<h2>Bước 3. Cài đặt các phụ thuộc và định cấu hình Kubernetes</h3>
<p>
Trong phần này, chúng tôi sẽ cài đặt các gói Docker và Kubernetes vào cả 3 node, sau đó kích hoạt chúng hoạt động. Ngoài ra, chúng tôi sẽ chuẩn bị các công cụ Kubernetes như <code>kubectl</code> và <code>kubeadm</code> khi chúng bắt buộc phải thiết lập.
</p>
<p><b><u>Quan trọng:</b> Các bước sau phải được thực hiện trong tất cả các node của bạn, bao gồm các node Master và Worker một cách song song.</u></p>

<h3>3.1. Cập nhật danh sách gói</h3>
<p>Bạn có thể cập nhật danh sách gói trong phiên bản Ubuntu của mình bằng lệnh sau:</p>

```commandline
sudo apt-get update
```

<h3>3.2. Cài đặt Docker</h3>
<p>Để cài đặt Docker trên phiên bản Ubuntu của bạn (tất cả các node), bạn nên sử dụng lệnh sau:</p>

```
sudo apt-get install docker.io
```

<h3>3.3. Kiểm tra phiên bản Docker</h3>
<p>Khi bạn cài đặt Docker trên các node của mình, bạn có thể dễ dàng kiểm tra phiên bản Docker vừa cài đặt để chắc chắn rằng nó đã được cài đặt thành công bằng lệnh sau:</p>

```
docker --version
```
</p>

<h3>3.4. Bắt đầu và kích hoạt Docker</h3>
Khi bạn đã cài đặt Docker ở bước trước, bạn nên bắt đầu và kích hoạt công việc theo loại và nhập các lệnh sau.
  
```commandline

# Đặt Docker để khởi chạy khi khởi động
sudo systemctl enable docker
# Xác minh rằng Docker hiện đang chạy
sudo systemctl status docker
# Nếu không chạy, hãy thử lệnh:
sudo systemctl start docker
```
</p>

<h2>Bước 4. Cài đặt Kubernetes trên Ubuntu 18.04</h2>
<p>
Để các phiên bản của chúng tôi được thiết lập chính xác, bên cạnh Docker, chúng tôi phải cài đặt dịch vụ Kubernetes. Kubernetes được sử dụng để sắp xếp các Docker image, tăng và giảm quy mô dựa trên nhu cầu thời gian thực và nó cũng có một số lợi ích. </p>

<h3>4.1. Thêm khóa ký (GPG)</h3>
<p>Vì bạn đang tải xuống Kubernetes từ một kho lưu trữ không chuẩn, điều cần thiết là phải đảm bảo rằng phần mềm là xác thực. Vì vậy, chúng tôi cần thêm khóa ký (với phần mở rộng <code>gpg</code>).<br>
Để thêm khóa GPG, hãy nhập và nhập comand sau:</p>
  
```commandline
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
```

<h3>4.2. Thêm kho phần mềm Kubernetes</h3>
<p>Khi bạn đã thêm khóa ký thành công, bạn phải thêm kho lưu trữ phần mềm bằng lệnh sau:</p>
  
```commandline
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

<h2>Bước 5. Chuẩn bị công cụ cài đặt Kubernetes</h2>
<p>Với bước này, chúng tôi sẽ chuẩn bị các công cụ Kubernetes cần thiết để bắt đầu và hoàn tất cấu hình các node.</p>

<h3>5.1. Cài đặt <i>kubeadm</i>, <i>kubelet</i>, và <i>kubectl</i></h3>
<p>Ở đây chúng ta sẽ cài đặt các công cụ <code>kubeadm</code>, <code>kubelet</code> và <code>kubectl</code>. Những công cụ này là cần thiết để khởi tạo và quản lý Cluster. Để cài đặt tất cả các công cụ này, hãy nhập và nhập các lệnh sau:</p>
  
```commandline
sudo apt-get install kubeadm kubelet kubectl
```

<h3>5.2. Kiểm tra xem <i>kubeadm</i>, <i>kubelet</i> và <i>kubectl</i> có đang bị treo không</h3>
<p>Khi chúng ta đã cài đặt các công cụ <i>kubeadm</i>, <i>kubelet</i> và <i>kubectl</i>, bây giờ chúng ta cần đảm bảo rằng các công cụ này đang hoạt động. Nói cách khác, chúng ta cần kiểm tra xem chúng có đang bị tạm dừng hay không. Chúng ta có thể làm điều đó bằng lệnh sau:</p>

```commandline
sudo apt-mark hold kubeadm kubelet kubectl  
```

<p>Ở đầu ra, bạn sẽ thấy ba dòng văn bản cho biết rằng các công cụ này hiện đang bị tạm dừng. Nếu có, hãy chuyển sang bước tiếp theo.</p>

<h3>5.3. Kiểm tra phiên bản <i>kubeadm</i></h3>
  
<p>Khi bạn hoàn thành bước trước đó, hãy chắc chắn rằng quá trình cài đặt đã thành công. Để thực hiện, hãy xác minh quá trình cài đặt và kiểm tra phiên bản <code>kubeadm</code> đã cài đặt bằng lệnh sau: </p>
  
```commandline
kubeadm version
```
Nếu bạn đã thực hiện đúng tất cả các bước trước đó, sẽ không có bất kỳ lỗi nào phát sinh. Bạn sẽ thấy một số dữ liệu phiên bản và siêu dữ liệu liên quan đến phiên bản Kubenetes của mình.
</p>

<h2>Bước 6. Triển khai Kubernetes</h2>
<p>Với phần này, chúng ta sẽ bắt đầu quá trình triển khai Kubernetes mới cài đặt trên các phiên bản EC2 cho tất cả các node. Tiếp tục thực thi các lệnh sau cho cả 3 node.</p>

<h3>6.1.Vô hiệu hóa bộ nhớ trao đổi từ mỗi node</h3>
<p>Hoán đổi là không gian đĩa cứng được sử dụng làm RAM. Nó (nói một cách tương đối) rất chậm, nhưng giúp máy tính không bị treo khi chúng đang cố xử lý nhiều dữ liệu hơn mức RAM của chúng có thể xử lý. Trước khi khởi tạo mạng Kubernetes nội bộ trên các phiên bản của chúng tôi, bạn phải tắt bộ nhớ trao đổi khỏi mỗi node bằng lệnh sau:</p>

```commandline
sudo swapoff -a
```
Nếu không có bất kỳ lỗi nào xảy ra từ phía bạn, hãy tiếp tục.
</p>

  <h3>6.2. Đặt tên máy chủ cho mỗi node</h3>
Ở đây chúng ta cần đặt tên máy chủ có ý nghĩa cho mỗi node. Mục đích của bước này là để dễ dàng nhận ra node nào là node Master và node nào là node Worker trong quy trình triển khai của chúng tôi. Bạn nên đặt tên máy chủ cho từng trường hợp riêng biệt:
  
<ul>
  <li>Cho node <b>Master</b>: <code>sudo hostnamectl set-hostname master-node</code></li>
  <li>Cho node <b>Worker</b> 1: <code>sudo hostnamectl set-hostname worker-1</code></li>
  <li>Cho node <b>Worker</b> 2: <code>sudo hostnamectl set-hostname worker-2</code></li>
</ul>
Bạn có thể đặt tên máy chủ theo ý muốn, một khi không có bất kỳ quy tắc nghiêm ngặt nào về điều đó. Chỉ cần lưu ý rằng những cái tên phải có ý nghĩa, nó sẽ giúp cuộc sống của bạn dễ dàng hơn sau này.
</p>

<h2>Bước 7. Giải pháp thay thế để tránh sự không phù hợp giữa các trình điều khiển Docker</h3>
  
   <b>Quan trọng</b>: bước này là tùy chọn vì một số trường hợp phát sinh lỗi do sự không khớp giữa các trình điều khiển đã cài đặt trong các bước tiếp theo. Tôi khuyên bạn nên áp dụng giải pháp thay thế này để tránh những vấn đề như vậy. Bạn phải triển khai giải pháp thay thế này trong tất cả các node của mình.
  
<p>Ý tưởng chính của bước giải quyết này là tạo tệp <i>daemon.json</i> với nội dung được chỉ định bên trong thư mục <i>/etc/docker/</i> để tránh sự không khớp giữa các trình điều khiển và tạo Kubernetes chạy khỏe. Ở bước này, ít nhất bạn phải làm quen với một số lệnh Linux, chẳng hạn như <code>vi</code> và <code>touch</code>.</p>

<h3>7.1. Tạo một file<i>daemon.json</i></h3>

<p>Trước tiên, bạn cần điều hướng đến thư mục <i>/etc/docker/</i> từ vị trí hiện tại của bạn trên máy chủ. Sau đó, bạn nên tạo một tệp JSON mới bằng lệnh <code>touch</code> như sau.</p>
  
```commandline
cd ../../etc/docker
sudo touch daemon.json
```
  
  Khi bạn đã tạo <i>daemon.json</i>, bạn phải điền vào tệp này với nội dung được chỉ định bên trong như bên dưới. Trước hết, mở tệp JSON này để chỉnh sửa bằng lệnh <code>vi</code>:
  
```commandline
sudo vi daemon.json 
```
  
Khi bạn đã vào không gian chỉnh sửa, hãy nhập (hoặc sao chép và dán) nội dung này:
  
```commandline
{
"exec-opts": ["native.cgroupdriver=systemd"]
} 
```
  
Đảm bảo rằng bạn đã dán nội dung này chính xác và thoát bằng cách lưu tệp bằng phím
 <code>:wq</code>.
</p>

<h3>7.2. Đặt lại dịch vụ Docker và <i>kubeadm</i></h3>
<p>Bây giờ, bạn cần khởi động lại dịch vụ Docker của mình và khởi tạo <code>kubeadm</code> trong điều kiện mới bằng các lệnh sau:

```commandline
sudo systemctl restart docker
sudo kubeadm reset
```
  
Nếu không có lỗi nào phát sinh, bạn đã sẵn sàng khởi tạo mạng Kubernetes của mình trên node Master.
</p>

<h2>Bước 8. Khởi tạo mạng Kubernetes Pod</h2>
Bước này khá lớn so với các bước còn lại, vì vậy hãy sẵn sàng thực hiện các bước tiếp theo một cách cẩn thận mà không bị gián đoạn. Bước này chỉ được thực hiện trong node Master. Với bước này, bạn sẽ tạo mạng Pod trong node Master của mình, mạng này cần thiết để Cluster giao tiếp với các node của nó.

<h3>8.1. Thiết lập Mạng Pod cho node Master</h3>

<p>Tôi nghĩ bước này là bước quan trọng nhất và là câu trả lời cho câu hỏi - <i>có phải tất cả các bước chúng ta thực hiện trước đây đều đúng</i>? Vì vậy, bạn có thể khởi tạo Kubernetes Pod Network trên node Master của mình bằng lệnh sau:</p>
  
```commandline
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
  
Việc thực hiện lệnh này có thể mất khoảng 2-3 phút. Cuối cùng, bạn sẽ nhận được một đầu ra khá lớn với lệnh <code>join</code> ở cuối (chúng ta sẽ sử dụng nó trong bước tiếp theo để kết nối các node Worker với node Master) và một số lệnh cơ bản ở phần đầu của thông điệp đầu ra. Tôi thực sự khuyên bạn nên sao chép toàn bộ thông báo đầu ra để sử dụng thêm. Ví dụ về đầu ra cho lệnh được thực hiện dưới đây:
</p>

```commandline
Your Kubernetes control-plane has initialized successfully!
To start using your cluster, you need to run the following as a regular user:
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
Alternatively, if you are the root user, you can run:
  export KUBECONFIG=/etc/kubernetes/admin.conf
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
Then you can join any number of worker nodes by running the following on each as root:
kubeadm join 172.31.37.224:6443 --token 2jcb53.krt1i08yljnnkyqb \
	--discovery-token-ca-cert-hash sha256:c6ef8a3b52fa8cc22f8933f502b61ce56b0ec135af9c5d1b503c8d1876a1a961 
```

<p>Kết quả đầu ra này cung cấp cho bạn rất nhiều hướng dẫn về những việc cần làm tiếp theo. Giữ mã thông báo <code>join</code> thật cẩn thận, chúng tôi sẽ sử dụng nó để nối các node Worker với node Master.</p>

<h3>8.2. Tạo một thư mục cho node Master đã khởi tạo</h3>

<p>Như đầu ra được tạo cho lệnh <i>init</i> gợi ý, trước hết sau đó, chúng ta cần tạo một thư mục đặc biệt trong node Master, chỉ cần sao chép và dán các dòng từ đầu ra, như sau: </p>

```commandline
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

<p>Nếu không có bất kỳ lỗi nào phát sinh, bạn đã tạo thành công một thư mục bắt buộc cho Cluster.</p>

<h3>8.3. Set up a virtual network for comunication across nodes (<i>flannel</i>)</h3>

<p>Như <a href="https://kubernetes.io/docs/concepts/cluster-administration/networking/">tài liệu Kubernetes chính thức</a> cho biết, Flannel là một mạng lớp phủ rất đơn giản đáp ứng các yêu cầu của Kubernetes . Nhiều người đã báo cáo thành công với Flannel và Kubernetes. Nói cách khác, Flannel là một cách đơn giản và dễ dàng để định cấu hình kết cấu mạng lớp 3 được thiết kế cho Kubernetes.
</p>

<p>Bạn có thể khởi tạo mạng <i>flannel</i> mới cho Kubernetes Cluster của mình bằng lệnh sau:</p>

```commandline
sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

<p>Tệp <i>kube-flannel.yml</i> đã tải chứa tất cả các hướng dẫn để cài đặt mạng này trên Cluster của bạn.</p>

<h3>8.4. Kiểm tra trạng thái pod</h3>
<p>Pods là đơn vị thực thi nhỏ nhất trong Kubernetes. Một pod đóng gói một hoặc nhiều ứng dụng. Về bản chất, các pod là tạm thời, nếu một pod (hoặc node mà nó thực thi) bị lỗi, Kubernetes có thể tự động tạo một bản sao mới của pod đó để tiếp tục hoạt động. Pod bao gồm một hoặc nhiều containers (chẳng hạn như containers Docker) (<a href="https://www.vmware.com/topics/glossary/content/kubernetes-pods">nguồn</a>).</p>

<p>Khi bạn đã khởi tạo thành công mạng <i>flannel</i> mới trên Cluster Kubernetes của mình, đây là thời điểm tốt để kiểm tra trạng thái pod. Bạn có thể làm điều đó bằng lệnh sau:</p>

```commandline
kubectl get pods --all-namespaces
```

<p>Trên đầu ra được tạo, bạn sẽ thấy rằng tất cả các pod của bạn trên node Master đang chạy.</p>
<h3>8.5. Tham gia các node Worker với một node Master</h3>
<p>Đây là (gần như) bước cuối cùng trong hướng dẫn này. Vì vậy, như đầu ra <code>init</code> đề xuất, chúng ta cũng cần kết nối hai (hoặc nhiều) node Worker của mình với node Master theo cách chính xác, với mã thông báo <code>join</code> được tạo. Đây là dòng cuối cùng trong đầu ra được đề cập, bắt đầu bằng <code>kubeadm join 172.31.37...</code>.</p>
<p>Tôi khuyên bạn nên xóa <code>\</code> khỏi lệnh và làm điều này để tạo một dòng thay vì hai dòng. Do đó, bạn sẽ thấy cấu trúc lệnh tương tự như sau:</p>

```commandline
kubeadm join 172.31.37.224:6443 --token 2jcb53.krt1i08yljnnkyqb --discovery-token-ca-cert-hash sha256:c6ef8a3b52fa8cc22f8933f502b61ce56b0ec135af9c5d1b503c8d1876a1a961 
```

<p>Hãy chạy đi! Do đó, bạn sẽ nhận được kết quả cho biết rằng các node Worker của bạn hiện được kết nối với node Master.</p>

<h2>Bước 9. Kiểm tra giai đoạn cuối</h2>

<p>Nếu bạn đang ở bước này, tôi cho rằng bạn đã hoàn tất thành công tất cả các bước trước đó. Để đảm bảo rằng mọi thứ đều ổn, hãy kiểm tra trạng thái cuối cùng của cluster bằng lệnh sau:</p>

```commandline
sudo kubectl get nodes
```


<p>Đầu ra của phần này phải là danh sách các node, cho biết rằng chúng (<i>Master</i>, <i>Worker-1</i> và <i>Worker-2</i> đang chạy hoặc chúng đang <i>đang chạy</i>, bao gồm cả <i>VERSION</i> và <i>AGE</i> Nếu bạn thấy điều đó, chúc mừng bạn đã hoàn thành hướng dẫn lớn này!</p>


---

<h2>Video hướng dẫn này hiện có - GIẢI THÍCH TẤT CẢ CÁC BƯỚC!</h2>
<p>Nếu bạn thích định dạng học bằng video thay vì đọc hướng dẫn của Readme hoặc muốn kết hợp cả hai, thì đây, hãy thưởng thức video được chuẩn bị kỹ lưỡng này trên Youtube!
<p>

<a href="https://youtu.be/SQJxYhfzznU">
  <img src="" alt="Setup Kubernetes on Ubuntu 18.04 - video tutoriall">
</a></p>
</p>

---
<p> Nguồn tham khảo chính <a href="https://github.com/vb100/Deploy_Kubernetes_AWS_EC2.git">
  Github</a></p>