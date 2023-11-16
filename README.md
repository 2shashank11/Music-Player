# Music-Player-

### Main.java
```
package com.kensoftph.javafxmedia;

import javafx.application.Application;
import javafx.fxml.FXMLLoader;
import javafx.scene.Scene;
import javafx.stage.Stage;

import java.io.IOException;

public class Main extends Application {
    @Override
    public void start(Stage stage) throws IOException {
        FXMLLoader fxmlLoader = new FXMLLoader(Main.class.getResource("media_player.fxml"));
        Scene scene = new Scene(fxmlLoader.load());
        stage.setTitle("JavaFX MediaPlayer!");
        stage.setScene(scene);
        stage.show();
    }

    public static void main(String[] args) {
        launch();
    }
}
```

### MediaPlayerController.java
```
package com.kensoftph.javafxmedia;

import java.io.File;
import java.util.LinkedList;
import java.util.List;
import java.util.Queue;

import javafx.beans.property.SimpleStringProperty;
import javafx.beans.property.StringProperty;
import javafx.event.ActionEvent;
import javafx.fxml.FXML;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.Label;
import javafx.scene.control.Slider;
import javafx.scene.input.MouseEvent;
import javafx.scene.media.Media;
import javafx.scene.media.MediaPlayer;
import javafx.scene.media.MediaView;
import javafx.stage.FileChooser;
import javafx.util.Duration;
import javafx.scene.image.Image;
import javafx.scene.image.ImageView;

public class MediaPlayerController {

    @FXML
    private Button btnPlay;

    @FXML
    private Label lblDuration;

    @FXML
    private MediaView mediaView;

    @FXML
    private Slider slider;

    @FXML
    private Button btnNext;

    @FXML
    private Button btnPrevious;

    @FXML
    private ImageView albumCoverImageView;
    @FXML
    private Label lblCurrentSong;

    private StringProperty currentSongName = new SimpleStringProperty("");


    private Media media;
    private MediaPlayer mediaPlayer;
    private final Queue<File> mediaQueue = new LinkedList<>(); // Queue to store media files
    private boolean isPlayed = false;

    @FXML
    void btnPlay(MouseEvent event) {
        if (mediaPlayer != null) {
            if (mediaPlayer.getStatus() == MediaPlayer.Status.PLAYING) {
                mediaPlayer.pause();
                btnPlay.setText("Play");
                isPlayed = false;
            } else {
                mediaPlayer.play();
                btnPlay.setText("Pause");
                isPlayed = true;
            }
        } else {
            playNextMedia();
        }
    }


    @FXML
    void btnStop(MouseEvent event) {
        btnPlay.setText("Play");
        mediaPlayer.stop();
        isPlayed = false;
    }

    @FXML
    void selectMedia(ActionEvent event) {
        FileChooser fileChooser = new FileChooser();
        fileChooser.setTitle("Select Media Files");
        List<File> selectedFiles = fileChooser.showOpenMultipleDialog(null);

        if (selectedFiles != null && !selectedFiles.isEmpty()) {
            mediaQueue.addAll(selectedFiles);
            playNextMedia();
        }
    }

    @FXML
    void btnNext(ActionEvent event) {
        mediaPlayer.stop();
        playNextMedia();
    }


    private void playNextMedia() {
        if (!mediaQueue.isEmpty()) {
            File selectedFile = mediaQueue.poll();
            String url = selectedFile.toURI().toString();

            String filename = selectedFile.getName();
            int dotIndex = filename.lastIndexOf(".");
            if (dotIndex > 0) {
                filename = filename.substring(0, dotIndex);
            }

            currentSongName.set(filename); // Update the current song name
            media = new Media(url);

            mediaPlayer = new MediaPlayer(media);

            mediaPlayer.setOnReady(() -> {

                Duration totalDuration = media.getDuration();
                slider.setMax(totalDuration.toSeconds());
                lblDuration.setText("Duration: 00 / " + (int) totalDuration.toSeconds());
                mediaPlayer.play();
                isPlayed = true;
                btnPlay.setText("Pause");
            });

            mediaView.setMediaPlayer(mediaPlayer);
            btnPlay.setText("Play");

            mediaPlayer.currentTimeProperty().addListener((observableValue, oldValue, newValue) -> {
                slider.setValue(newValue.toSeconds());
                lblDuration.setText("Duration: " + (int) slider.getValue() + " / " + (int) media.getDuration().toSeconds());
            });

            Scene scene = mediaView.getScene();
            mediaView.fitWidthProperty().bind(scene.widthProperty());
            mediaView.fitHeightProperty().bind(scene.heightProperty());
            lblCurrentSong.textProperty().bind(currentSongName);

        }
    }

    @FXML
    private void sliderPressed(MouseEvent event) {
        mediaPlayer.seek(Duration.seconds(slider.getValue()));
    }
}
```

### media_player.fxml
```
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.Insets?>
<?import javafx.scene.control.Button?>
<?import javafx.scene.control.Label?>
<?import javafx.scene.control.Slider?>
<?import javafx.scene.image.ImageView?>
<?import javafx.scene.layout.BorderPane?>
<?import javafx.scene.layout.HBox?>
<?import javafx.scene.layout.VBox?>
<?import javafx.scene.media.MediaView?>

<BorderPane maxHeight="-Infinity" maxWidth="-Infinity" minHeight="-Infinity" minWidth="-Infinity" prefHeight="216.0" prefWidth="850.0" xmlns="http://javafx.com/javafx/21" xmlns:fx="http://javafx.com/fxml/1" fx:controller="com.kensoftph.javafxmedia.MediaPlayerController">
   <center>
      <MediaView fx:id="mediaView" fitHeight="200.0" fitWidth="200.0" BorderPane.alignment="BOTTOM_CENTER" />

   </center>
   <bottom>
      <VBox prefHeight="120.0" prefWidth="850.0" BorderPane.alignment="CENTER">
         <children>
            <HBox alignment="CENTER" prefHeight="43.0" prefWidth="850.0">
               <children>
                  <Slider fx:id="slider" onMousePressed="#sliderPressed" HBox.hgrow="ALWAYS" />
               </children>
               <padding>
                  <Insets left="15.0" right="15.0" />
               </padding>
            </HBox>
            <HBox alignment="CENTER" prefHeight="100.0" prefWidth="200.0" spacing="10.0">
               <children>
                  <Label fx:id="lblCurrentSong" />
                  <Button mnemonicParsing="false" onAction="#selectMedia" text="Select Media" />
                  <Button fx:id="btnPlay" mnemonicParsing="false" onMouseClicked="#btnPlay" text="Play" />
                  <Button mnemonicParsing="false" onMouseClicked="#btnStop" text="Stop" />
                  <Button fx:id="btnNext" layoutX="220.0" layoutY="210.0" onAction="#btnNext" text="Next" />
                  <Label fx:id="lblDuration" text="Duration: 00 / 00" />
               </children>
            </HBox>
         </children>
      </VBox>
   </bottom>
</BorderPane>
```
