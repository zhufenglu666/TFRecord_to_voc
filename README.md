# TFRecord_to_voc

import os
import xml.etree.ElementTree as ET


def convert(size, box):
    x_center = (box[0] + box[1]) / 2.0
    y_center = (box[2] + box[3]) / 2.0
    x = x_center / size[0]
    y = y_center / size[1]
    w = (box[1] - box[0]) / size[0]
    h = (box[3] - box[2]) / size[1]
    return x, y, w, h


def convert_single_annotation(xml_file, save_txt_file, classes):
    try:
        tree = ET.parse(xml_file)
        root = tree.getroot()
        size = root.find('size')
        w = int(size.find('width').text)
        h = int(size.find('height').text)

        with open(save_txt_file, 'w') as out_txt_f:
            for obj in root.iter('object'):
                difficult = obj.find('difficult').text
                cls = obj.find('name').text
                if cls not in classes or int(difficult) == 1:
                    continue
                cls_id = classes.index(cls)
                xmlbox = obj.find('bndbox')
                b = (float(xmlbox.find('xmin').text), float(xmlbox.find('xmax').text),
                     float(xmlbox.find('ymin').text), float(xmlbox.find('ymax').text))
                bb = convert((w, h), b)
                out_txt_f.write(f"{cls_id} {' '.join(map(str, bb))}\n")
    except (ET.ParseError, FileNotFoundError) as e:
        print(f"Error processing {xml_file}: {e}")


def convert_annotation(xml_files_path, save_txt_files_path, classes):
    xml_files = [f for f in os.listdir(xml_files_path) if f.endswith('.xml')]
    print(xml_files)
    for xml_name in xml_files:
        print(xml_name)
        xml_file = os.path.join(xml_files_path, xml_name)
        out_txt_path = os.path.join(save_txt_files_path, xml_name.split('.')[0] + '.txt')

        if not os.path.exists(save_txt_files_path):
            os.makedirs(save_txt_files_path)

        convert_single_annotation(xml_file, out_txt_path, classes)


if __name__ == "__main__":
    classes1 = ['1']
    xml_files1 = r'zhuzi/data'
    save_txt_files1 = r'zhuzi/data'

    convert_annotation(xml_files1, save_txt_files1, classes1)
