# AVCapturePhotoOutput
An example on how to use AVCapturePhotoOutput on macOS with Swift 5.0 (with XCode Project)

```
//
//  ViewController.swift
//

import AVFoundation
import Cocoa
import AppKit

class ViewController: NSViewController, AVCapturePhotoCaptureDelegate {
    // MARK: - Properties
    var previewLayer: AVCaptureVideoPreviewLayer?
    var captureSession: AVCaptureSession?
    var captureConnection: AVCaptureConnection?
    var cameraDevice: AVCaptureDevice?
    var photoOutput: AVCapturePhotoOutput?
    var mouseLocation: NSPoint { NSEvent.mouseLocation }
    
    // MARK: - LyfeCicle
    override func viewDidLoad() {
        super.viewDidLoad()
    
        // Do any additional setup after loading the view.
        prepareCamera()
        startSession()
    }
    
    // MARK: - UtilityFunctions
    func truncateNumber(n: CGFloat) -> CGFloat {
        return floor(1000000 * n) / 1000000;
    }
    
    func moveMouseToRandomScreenPoint() {
        let s = NSScreen.main
        let sFrame = s?.frame
        let sWidth = Float((sFrame?.size.width)!)
        let sHeight = Float((sFrame?.size.height)!)
        let r = Float(Float.random(in: 0..<1))
        let rSWidth = CGFloat(r * sWidth)
        let rSHeight = CGFloat(r * sHeight)
        
        let screenPoint = CGPoint(x: rSWidth, y: rSHeight)
        
        moveMouseToScreenPoint(screenPoint: screenPoint)
    }
    
    func moveMouseToScreenPoint(screenPoint: CGPoint) {
        let moveEvent = CGEvent(mouseEventSource: nil, mouseType: .mouseMoved, mouseCursorPosition: screenPoint, mouseButton: .left)
        moveEvent?.post(tap: .cgSessionEventTap);
    }

    override var representedObject: Any? {
        didSet {
        // Update the view, if already loaded.
        }
    }
    
    @IBAction func button(_ sender: Any) {
        moveMouseToRandomScreenPoint()
        capturePhoto()
    }
    
    func startSession() {
        if let videoSession = captureSession {
            if !videoSession.isRunning {
                videoSession.startRunning()
            }
        }
    }
    
    func stopSession() {
        if let videoSession = captureSession {
            if videoSession.isRunning {
                videoSession.stopRunning()
            }
        }
    }
    
    internal func photoOutput(_ output: AVCapturePhotoOutput, willBeginCaptureFor resolvedSettings: AVCaptureResolvedPhotoSettings) {
        print("willBeginCaptureFor")
    }
    
    internal func photoOutput(_ output: AVCapturePhotoOutput, didFinishProcessingPhoto photo: AVCapturePhoto, error: Error?) {
        print("didFinishProcessingPhoto")
        print(photo)
    }
    
    func capturePhoto() {
        print(captureConnection?.isActive)
        let photoSettings = AVCapturePhotoSettings()
        photoOutput?.capturePhoto(with: photoSettings, delegate: self)
    }
    
    func prepareCamera() {
        photoOutput = AVCapturePhotoOutput()
        captureSession = AVCaptureSession()
        captureSession!.sessionPreset = AVCaptureSession.Preset.photo
        previewLayer = AVCaptureVideoPreviewLayer(session: captureSession!)
        previewLayer!.videoGravity = AVLayerVideoGravity.resizeAspectFill
        do {
            let deviceDiscoverySession = AVCaptureDevice.DiscoverySession(deviceTypes: [AVCaptureDevice.DeviceType.builtInWideAngleCamera], mediaType: AVMediaType.video, position: AVCaptureDevice.Position.front)
            let cameraDevice = deviceDiscoverySession.devices[0]
            let videoInput = try AVCaptureDeviceInput(device: cameraDevice)
            captureSession!.beginConfiguration()
            if captureSession!.canAddInput(videoInput) {
                print("Adding videoInput to captureSession")
                captureSession!.addInput(videoInput)
            } else {
                print("Unable to add videoInput to captureSession")
            }
            if captureSession!.canAddOutput(photoOutput!) {
                captureSession!.addOutput(photoOutput!)
                print("Adding videoOutput to captureSession")
            } else {
                print("Unable to add videoOutput to captureSession")
            }
            captureConnection = AVCaptureConnection(inputPorts: videoInput.ports, output: photoOutput!)
            captureSession!.commitConfiguration()
            if let previewLayer = previewLayer {
                if ((previewLayer.connection?.isVideoMirroringSupported) != nil) {
                    previewLayer.connection?.automaticallyAdjustsVideoMirroring = false
                    previewLayer.connection?.isVideoMirrored = true
                }
                previewLayer.frame = view.bounds
                view.layer = previewLayer
                view.wantsLayer = true
            }
            captureSession!.startRunning()
        } catch {
            print(error.localizedDescription)
        }
    }
}

```
