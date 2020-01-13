<script>
import ioClient from "socket.io-client";

let IOClient = ioClient.connect("http://localhost:3000");

console.log(IOClient.disconnected);

export default IOClient;
</script>
