import os
from time import time

import Image



# if set to 1 it recognizes only exact symbols
# for the 2nd captcha must be at least 1000
# bigger number, the more strange quessing 
MAX_MARK = 3000
###############

def to_log(message_str, username=''):
    if username:
        filepath = str(username) + '_log.txt'
    else: 
        filepath = 'log.txt'
    if os.path.exists(filepath):
        mode = 'a'
    else:
        mode = 'w'
    message_str = "\n%s: "%str(time()) + message_str
    for i in xrange(5):
        i += 1
        if not i==1: sleep(1)
        try:
            with open(filepath, mode) as f:
                f.write(message_str)
            break
        except IOError, e:
            print "Cannot write %s:"%filepath, e
            
def load_samples(case):
    alphabet_path = os.path.join('alphabets',case)
    if not os.path.exists(alphabet_path): os.makedirs(alphabet_path)
    
    alphabet = (case,)
    for item in os.listdir(alphabet_path):
        path_name = os.path.join(alphabet_path, item)
        if os.path.isdir(path_name): continue
        
        char = item.split('_')[0]
        if len(char) == 1:            
            s = Symbol(path_name, char)
            alphabet = alphabet + (s,)
        else:
            to_log("%s ignored."%path_name)
                
    if not (case=='3') and len(alphabet)<20:
        print "Something wrong. The collection of letters has loaded"
        print "less than 20 samples."
        return None
    return alphabet

class Symbol:
    """
    Represents the image data of a symbol
    (some sample from a collection with some offset)
    propertires: filename, data, width, height, char, offset
    """
    def __init__(self, pathname, char):
        im = Image.open(pathname)
        im = im.convert('L') # to [0-255] numbers in data
        head, self.filename = os.path.split(pathname)
        self.data = im.load()
        self.width, self.height = im.size
        
        self.pydata = []
        for x in xrange(self.width):
            self.pydata.append([])
            for y in xrange(self.height):
                self.pydata[x].append(self.data[x,y])
        self.data = self.pydata
        
        self.char = char
        self.captcha_offset = (0,0)

    def calculate_mark(self, captcha_data, case):
        """
        Calculates a mark for symbol (some sample with some offset).
        """
        mark = 0
        o_x, o_y = self.captcha_offset
        for x in xrange(self.width):
            for y in xrange(self.height):
                # if mark is too big already then BREAK
                if mark > MAX_MARK: return "big"
                difference = abs(self.data[x][y]-captcha_data[x + o_x, y + o_y])
                # divided by 10, and squared
                difference = (difference/10)*(difference/10)
                mark = mark + difference
        return mark
    
def solve_captcha(captcha_path, alphabet):
    # preparing    
    if alphabet=='1' or alphabet=='2' or alphabet=='3' :
        case = alphabet
    else:
        case = alphabet[0]
        
    try:
        captcha_im = Image.open(captcha_path)
    except IOError,e:
        msg = "Cannot open a captcha image. %s"%e
        print msg; to_log(msg)
        return None
    
    #checks that it is a small image file
    w, h = captcha_im.size
    if (w + h) > 400:
        msg = "Too big captcha image. %s %s"%(captcha_im.size,captcha_path)
        to_log(msg)
        raise Exception(msg)
    
    # alphabet's paths
    alphabet_path = os.path.join('alphabets',case)
    new_dir_path = os.path.join(alphabet_path,"new")
    if not os.path.exists(new_dir_path): os.makedirs(new_dir_path)
    new_name = str(time())[-2:]
    
    # check alphabet is gotten
    if type(alphabet)==type(('l',)):
        alphabet = alphabet[1:]
    else: #loads alphabet
        if not os.path.exists(alphabet_path): os.makedirs(alphabet_path)
        alphabet = ()
        for item in os.listdir(alphabet_path):
            path_name = os.path.join(alphabet_path, item)
            if os.path.isdir(path_name): continue
            char = item.split('_')[0]
            if len(char) == 1:            
                s = Symbol(path_name, char)
                alphabet = alphabet + (s,)
            else:
                to_log("%s ignored."%path_name)
        if not (case=='3') and len(alphabet)<20:
            print "Something wrong. The collection of letters has loaded"
            print "less than 20 samples."
            return None
        
    #make greyscale
    captcha_im = captcha_im.convert('L')
    
    #save for development
    new_img_path = os.path.join(new_dir_path, 'start_'+new_name)
    captcha_im.save(new_img_path+'.bmp')
    
    #cut the left empty piece of the image
    captcha_data = captcha_im.load()
    empty_x = 0
    for x in xrange(10):
        is_different = False
        for y in xrange(1,h):
            if not captcha_data[x,y]==captcha_data[x,y-1]:
                is_different = True
                break
        if is_different: break
        else: empty_x = x
    if empty_x>0:
        box = (empty_x, 0, w, h)
        captcha_im = captcha_im.crop(box)
    
    # general idea: recognizes the left area of data, cuts it, then to repeat
    recognized_str = ''
    while True:
        captcha_data = captcha_im.load()
        
        # compare to every letter with offsets:(0,0,4,data.height - symbol.height)
        symbols_marks = []
        mark_0_gotted = False
        
        for symbol in alphabet:
            possible_x = min (4,  captcha_im.size[0] - symbol.width)
            #the first letter
            #if len(recognized_str)==0: possible_x = 10
            possible_y = captcha_im.size[1] - symbol.height
            if possible_y<0 or possible_x<0 : # this symbol is too large
                continue
            
            for o_x in xrange(possible_x + 1):
                for o_y in xrange(possible_y + 1):
                    # for every pixels: mark + difference (possible, ^2)
                    symbol.captcha_offset = (o_x, o_y)
                    mark = symbol.calculate_mark(captcha_data, case)
                    
                    # appends (mark, symbol) to a list of candidates
                    if not mark == "big" :
                        ##to_log("%s with offset (%s, %s) riched %s"%(symbol.filename, o_x, o_y, mark))
                        symbols_marks.append((mark, symbol, o_x, o_y))
                    if mark == 0:
                        #NOTE if 2 equal marks, bigger symbol is better
                        mark_0_gotted = True
                        break
                if mark_0_gotted: break
            if mark_0_gotted: break
        # saves the minimal mark symbol from the list of candidates
        min_mark = MAX_MARK
        for i in xrange(len(symbols_marks)):
            if symbols_marks[i][0] < min_mark:
                min_mark = symbols_marks[i][0]
                min_mark_i = i
        if min_mark==MAX_MARK:
            print "Captcha isn't solved. Saved to "+alphabet_path+"/new/"
            to_log("Captcha isn't solved. Saved to "+alphabet_path+"/new/")
            
            new_img_path = os.path.join(new_dir_path, new_name)
            captcha_im.save(new_img_path+'.bmp')
            return None
        else:
            mark, recognized_symbol, o_x, o_y = symbols_marks[min_mark_i]
            # append symbol.char to main result string
            recognized_str += recognized_symbol.char
            
            print "With mark "+str(mark)+" '"+\
                  recognized_symbol.char+"' symbol guessed."
            
            if (len(recognized_str)>5 and case=='1') or \
            (len(recognized_str)>3 and case=='2') or\
            (len(recognized_str)>0 and case=='3'):
                if recognized_str=='1' and case=='3':
                    # cut symbol.width piece from left
                    box = (o_x + recognized_symbol.width, 0,
                           captcha_im.size[0], captcha_im.size[1])
                    captcha_im = captcha_im.crop(box)
                    # save
                    filepath = os.path.join(new_dir_path, new_name)+'.bmp'
                    captcha_im.save(filepath)
                    # solve tail
                    rank = solve_captcha(filepath, alphabet_3)
                    if rank=='0':
                        recognized_str = '10'
                    break
                else:
                    break
            
            # cut symbol.width piece from left
            box = (o_x + recognized_symbol.width - 1, 0,
                   captcha_im.size[0], captcha_im.size[1])
            captcha_im = captcha_im.crop(box)
        
    print "Result of solver: '"+recognized_str+"'"
    return recognized_str

if __name__=="__main__":
    alphabet_3 = load_samples('3')
    rank = solve_captcha('10von10.png', alphabet_3)
    print rank

