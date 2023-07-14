# PencilKit-Simple
This is the introductory version of another tutorial.
# The first part
![IMG_0307](https://github.com/S-way520/PencilKit-Simple/assets/95877651/eb5cf352-021c-4be7-a1db-49b6706c668f)
# The second part
Code file 1
```swift
import SwiftUI
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            DrawingView()
        }
    }
}
```
Code file 2
```swift
import SwiftUI
import PencilKit
struct DrawingView: View {
    @Environment(\.undoManager) private var undoManager
    @Environment(\.colorScheme) var colorScheme
    @State private var canvasView = PKCanvasView()
    @State private var toolPicker = PKToolPicker()
    @State var image: UIImage?
    @State var BackImage: UIImage?
    @State var newImage: UIImage?
    @State var Share = false
    func saveImage() -> Void {
        let image = canvasView.drawing.image(from: CGRect(x: 0, y: 0, width: 4000, height: 3000), scale: 1.0)
        let BackImage = UIImage(named: colorScheme == .light ?  "LightLine" : "BlackLine")
        UIGraphicsBeginImageContext(CGSize(width: 4000, height: 3000))
        BackImage!.draw(in: CGRect(x: 0, y: 0, width: 4000, height: 3000))
        image.draw(in: CGRect(x: 0, y: 0, width: 4000, height: 3000))
        let newImage = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        self.newImage = newImage
    }
    var body: some View {
        ZStack {
            CanvasView(canvasView: $canvasView, toolPicker: $toolPicker, savedImage: saveImage)
            VStack {
                HStack(spacing: 15){
                    Button(action: {self.Share.toggle()}){
                        Image(systemName: "square.and.arrow.up").font(.title2)
                    }
                    .sheet(isPresented: $Share){
                        ShareSheet(activityItems: [newImage as Any])
                            .presentationDetents([.large, .medium])
                    }
                    Button(action: {undoManager?.undo()}){
                        Image(systemName: "arrow.uturn.backward").font(.title2)
                    }
                    Button(action: {undoManager?.redo()}){
                        Image(systemName: "arrow.uturn.right").font(.title2)
                    }
                    Button(action: {canvasView.drawing = PKDrawing()}){
                        Image(systemName: "trash").font(.title2)
                    }
                }
                .background(Color(UIColor.systemBackground))
                .cornerRadius(5, antialiased: true)
                Spacer()
            }
            .padding()
            if Share {
                ShareSheet(activityItems: [image as Any])
                    .frame(width: 1, height: 1, alignment: .center)
            }
        }
    }
}
struct CanvasView: UIViewRepresentable {
    @Binding var canvasView: PKCanvasView
    @Binding var toolPicker: PKToolPicker
    let savedImage: () -> Void
    func makeUIView(context: Context) -> PKCanvasView {
        canvasView.tool = PKInkingTool(.pen, color: .blue, width: 5)
        canvasView.drawingPolicy = .default
        canvasView.delegate = context.coordinator //设置委托
        canvasView.contentSize = CGSize(width: 4000, height: 3000) //设置内容尺寸
        canvasView.showsHorizontalScrollIndicator = false //隐藏水平滚动条
        canvasView.showsVerticalScrollIndicator = false //隐藏垂直滚动条
        canvasView.backgroundColor = .clear //清除背景
        canvasView.becomeFirstResponder() //设置 canvasView 为第一响应视图
        toolPicker.setVisible(true, forFirstResponder: canvasView) //工具选择器的可见性
        toolPicker.addObserver(canvasView) //监听 canvesView 视图
        canvasView.isOpaque = true
        canvasView.addSubview(UIImageView(image: UIImage(named:"BackLine")))//背景图
        return canvasView
    }
    func updateUIView(_ canvasView: PKCanvasView, context: Context) {}
    func makeCoordinator() -> Coordinator {
        return Coordinator(canvasView: $canvasView, savedImage: savedImage)
    }
    class Coordinator: NSObject, PKCanvasViewDelegate , PKToolPickerObserver{//Save
        var canvasView: Binding<PKCanvasView>
        let savedImage: () -> Void
        init(canvasView: Binding<PKCanvasView>, savedImage: @escaping () -> Void) {
            self.canvasView = canvasView
            self.savedImage = savedImage
        }
        func canvasViewDrawingDidChange(_ canvasView: PKCanvasView) {
            if !canvasView.drawing.bounds.isEmpty { savedImage() }
        }
    }
}
struct ShareSheet: UIViewControllerRepresentable {//Share
    typealias Callback = (
        _ activityType: UIActivity.ActivityType?, 
        _ completed: Bool, 
        _ returnedItems: [Any]?, 
        _ error: Error? ) -> Void
    let activityItems: [Any]
    let applicationActivities: [UIActivity]? = nil
    let excludedActivityTypes: [UIActivity.ActivityType]? = nil
    let callback: Callback? = nil
    func makeUIViewController(context: Context) -> UIActivityViewController {
        let controller = UIActivityViewController(activityItems: activityItems, applicationActivities: applicationActivities)
        controller.excludedActivityTypes = excludedActivityTypes
        controller.completionWithItemsHandler = callback
        return controller
    }
    func updateUIViewController(_ uiViewController: UIActivityViewController, context: Context) {}
}
```
